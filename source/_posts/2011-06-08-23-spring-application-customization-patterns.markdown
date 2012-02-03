---
layout: post
title: "Spring Application Customization Patterns"
date: 2011-06-08 23:03
comments: true
categories: 
---

<p>Having performed numerous customizations of third party or upstream/vendor applications based on the Spring framework, I'd like to share some patterns I've identified. These patterns form a hierarchy based on preferability - if it is not possible to implement one approach you may have to back off, and try the next best, repeating in stepwise fashion. I am going to list them from least preferable to most preferable in order to walk you through the rationale behind each approach.</p>
<p>We will assume an application distributed in binary form, containing a Spring context with a service interface <strong>com.sample.ApplicationService</strong> and implementation <strong>com.sample.ApplicationServiceImpl</strong> defined as an <strong>"applicationService"</strong> bean which we want to override with some local customization.</p>
<p><strong>Direct class override</strong></p>
<p>The simplest and most direct approach is to simply create a new class named <strong>com.sample.ApplicationServiceImpl</strong> and arrange for it to appear in the classpath before the upstream implementation. If you have no other options (or are simply prototyping or debugging something) then this might work for you, but it cannot be considered much more than a hack.</p>
<p>There are numerous problems with this approach. First, while it may initially be superficially possible to contrive a classloader chain which results in your implementation being picked up first, this is brittle and may lead to very confusing results when it breaks in the future. You will either be duplicating the service implementation by copying and pasting, or otherwise lying about the pedigree of this class. This will certainly make debugging harder, and will also convey an immediate maintenance overhead. Imagine upstream diverging from the base you forked - you will now have to continually reconcile any future changes to upstream (this may not phase users of decentralized version control systems but is certainly endless and error-prone tedium when working with central SCM like SVN). If you hoped to submit your customizations for inclusion upstream, then overriding the class directly may complicate acceptance. If behavior has changed, tests may fail. Or the change may be rejected because it is not suitable as a default for all users.</p>
<p>A better approach would be to implement a separate class and submit this implementation as an <em>alternative</em> (with tests specific to this implementation of course).</p>
<p><strong>Explicit context inclusion</strong></p>
<p>In order to supply an alternate implementation, you will need to override the service bean defined in the application's Spring context (let's assume there <em>is</em> such a bean defined). Unless the application has provided some way for you to inject a context to override this bean you are still in the first situation: having to directly override a file (the Spring context this time) relying on classloader precedence. Let's assume the application has implemented a specific provision for you to specify your own Spring context for inclusion. It may look like this:</p>

``` java
String customContext = System.getProperty("customContext");

if (customContext == null) {
  context = new ClasspathApplicationContext(DEFAULT_CTX);
} else {
  context = new ClasspathApplicationContext(new String[] { DEFAULT_CONTEXT, customContext });
}

```

<p>In this situation you can construct a Spring context which redefines the application bean by name ("applicationService") and supply the context via a system parameter:

```
-DcustomContext=MySpecialCustomization.xml
```

<p>However this requires explicit, programmatic, support by the application.</p>
<p><strong>Automatic context inclusion</strong></p>
<p>A better approach for the application to implement is to take advantage of Spring's resource syntax to import a <strong>wildcard</strong> resource expression. For example:</p>
``` xml
<import resource="classpath*:com/sample/*Customization.xml">
```
<p>If placed in the application's context, this expression will cause the import of any contexts matching the glob <strong>"com/sample/*Customization.xml"</strong> in any classloader without any <em>additional coordination between application and implementer.</em></p>
<p>The <strong>"classpath*:" </strong>syntax tells Spring to look in <em>all classloaders</em>. See Spring's documentation for a better understanding of classpath resource expressions:</p>

<a href="http://static.springsource.org/spring/docs/2.5.x/reference/resources.html#resources-app-ctx-wildcards-in-resource-paths">http://static.springsource.org/spring/docs/2.5.x/reference/resources.html#resources-app-ctx-wildcards-in-resource-paths</a>

<p>If using Maven, for example, simply adding an additional dependency containing the customization (e.g. "com/sample/MySpecialCustomization.xml") to your project is sufficient to implement the override.</p>
<p>One theoretical downside is that it may be expensive for Spring to iterate through classloaders looking for matching contexts. This is complete speculation on my part, and given the maintenance burden and complexity of the other solutions, it is probably worth a (startup) performance penalty.</p>
<p><strong>Missing bean definition</strong></p>
<p>The above approaches will not work (directly) if you encounter the following situation: You wish to override a class which is not defined as a bean. It is likely this class itself is used (eventually) by a service, which means you need to walk the caller hierarchy to find affected services, and then resume with the above approaches <em>on those services</em>. You will need to override the services, providing implementations (ideally shallow subclasses) which override <em>the minimal number of methods</em>necessary to replace invocations of the old class with your new class.</p>
<p>For bonus points, make the aforementioned class <em>injectable</em>as a Spring bean, and submit this enhancement upstream. This will allow you to migrate your customization to a simple bean and throw away your unnecessary wrapper classes/beans.</p>
<p><strong>AOP magic</strong></p>
<p>With the approaches discussed so far, you ultimately still have to replace an entire service. If the application has been modularized sufficiently then this may not be a concern. However, it may be the case that replacing an entire service is heavy-handed, or that the functionality you need implemented is not necessarily best done by replacing a service implementation.</p>
<p>In these situations there is another trick up our sleeve. We can wrap the <strong><em>existing</em></strong> implementation bean with a Spring AOP (Aspect Oriented Programming) aspect. For example we can surgically override a single method on a large interface. Or we can apply common logic to all methods (e.g. caching). We can secretly mutate method arguments and return values. In some situations this is much more convenient than overriding a service (imaging having to override a large service simply to change arguments and delegate to the wrapper service...).</p>
<p>While this is the most loosely coupled approach, there are several downsides. First, AOP can be hard to understand and maintain. Secondly, AOP can complicate debugging as errors can now occur "between" known call frames, and behavior introduced dynamically via AOP will not be obvious to the external observer. Lastly, AOP can only be applied to services defined as beans.</p>
<p>For details on Spring AOP see:</p>

<a href="http://static.springsource.org/spring/docs/2.5.x/reference/aop.html">http://static.springsource.org/spring/docs/2.5.x/reference/aop.html</a>
<p><strong>Conclusion</strong></p>
<p>Well, despite the lack of code examples I hope that this post has been helpful. I may in the future construct a sample case that illustrates each of these approaches, and the amount of "maintenance" overhead they impose.</p>
<p>To sum up, I'd like to conclude with:</p>
<p>Direct class overriding: JUST DON'T DO IT</p>
<p>Automatic context inclusion: PLEASE implement this provision if you are writing an application or service in Spring which you anticipate users having to customize</p>
