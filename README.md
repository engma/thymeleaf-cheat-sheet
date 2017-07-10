Thymeleaf cheat sheet
==================
This is a cheat sheet to summarize all the main thymeleaf features and how to use them to kickstart you with thymeleaf.

-----------

## What is Thymeleaf


Thymeleaf is an engine that builds dynamic pages from templates that are written in XHTML with the help of some special attributes, so it is a **template engine**.

A template engine in Java is an engine that parses XHTML pages which contain special tags or attributes or syntax, those variables are bound to fields of beans on the server side, and resolves them into their actual values, then processes the page according to those values and builds a normal HTML page.

The templating engine will then resolve the values of the list of students that was given from the controller, and then use those values to replace the attributes and dummy text with actual data from the fetched list.

Thymeleaf is an **in-memory** template engine, so it does all of it's processing in memory, it builds a DOM that maps to the HTML of the page and binds the values to those fields which are displayed, and when values from server change the parsed fields and pages are updated accordingly, also it's caching is an in-memory caching system, meaning cache will invalidate with server restarts.

Thymeleaf is a template engine that relys mostly on **attributes** instead of **tags** as in JSP or JSF, this makes it testable in the browser directly without requiring a server to parse the special html tags, which eases the work between designer and developer, as they can both test the same page.

Those attributes are then translated and processed by Thymeleaf into normal HTML.
### How it works
`<p th:text="'Thymeleaf will display this'">text</p>`
Here thymeleaf will process the text inside the `th:text` attribute, and replace the contents of the `<p>` tag with it.
Thymeleaf works by replacing the contents of the tags that it's attributes are defined on
Another example is:

```html
<tr th:each="prod : ${prods}">
    <td th:text="${prod.name}">Onions</td>
    <td th:text="${prod.price}">2.41</td>
<tr>
```

Here thymeleaf will repeat the `<tr>` with the list of products, this is defined by the attribute `th:each`, it will also remove the dummy content in both the `<td>` tags, and replace them with the content that is evaluated from `th:text="${prod.name}"` and `th:text="${prod.price}"`.

_________

## Thymeleaf Layout Dialect

This dialect adds JSF-like template hierarchy to the Thymeleaf Engine, which makes it possible that you have templates extending other templates and overriding fragments of those parent templates that we open for extension. This is useful when you have a common layout that you want to apply to all your pages and views, for example a footer or a sidebar or a common CSS and JavaScript tags. 
To start you need the dependency:

```xml
<dependency>
    <groupId>nz.net.ultraq.thymeleaf</groupId>
    <artifactId>thymeleaf-layout-dialect</artifactId>
    <version>1.3.1</version>
</dependency>
```

Now you can add your common content in your parent laytout page, and then define the part that you want extending templates to substitute with their custom content in a **fragment**, then in extending templates you use this parent template as your **layout-decorator**, then override the **fragment** with your custom content. 

Example say we have a **main.html** which contains a navbar, a sidebar, and a footer, and it has the middle part empty waiting for content to be inserted in it, it then defines this middle part as follows

```html
<div class="container">
    <div layout:fragment="content" class="noborder">
    </div>
</div>
```

Then in the overriding template, for example **index.html**, we use `layout:decorator="main"` at the `<html>` tag, where **main** is the parent template to be extended.
Then in **index.html** we do this to override the fragment `content`

```html
<div layout:fragment="content">
      <p th:text="${template}">Should not be displayed</p>
</div>
```

This will override the fragments content withe the content in the `div` tag.

for more information, please check [this](https://github.com/ultraq/thymeleaf-layout-dialect)

____________

### Spring Integration
Thymeleaf has a spring integration project, that eases the integration of **Spring MVC** with thymeleaf as a template.

1. Add `thyemelaf-spring` dependency to your dependencies

	```xml
	<dependency>
	     <groupId>org.thymeleaf</groupId>
	     <artifactId>thymeleaf-spring4</artifactId>
	     <version>2.1.4.RELEASE</version>
	</dependency>
	```

2. Add this configuration to your spring servlet configuartion

	```xml
	<bean id="templateResolver"
		class="org.thymeleaf.templateresolver.ServletContextTemplateResolver">
		<property name="prefix" value="/WEB-INF/templates/" />
		<property name="suffix" value=".html" />
		<property name="templateMode" value="HTML5" />
	</bean>
	<bean id="templateEngine" class="org.thymeleaf.spring4.SpringTemplateEngine">
	     <property name="templateResolver" ref="templateResolver" />
	     <property name="additionalDialects">
		  <set>
		     <bean class="nz.net.ultraq.thymeleaf.LayoutDialect" />
		  </set>
	     </property>
	</bean>
	<bean class="org.thymeleaf.spring4.view.ThymeleafViewResolver">
	     <property name="templateEngine" ref="templateEngine" />
	</bean>
	```   

	This configuration will make the thymeleaf resolver the `ViewResolver` of Spring MVC, the `<property name="templateMode" value="HTML5" />` is particulary important, as it sets the mode of which thymeleaf should operate, we here specify the mode to be **HTML5**, which means that thymeleaf should produce valid HTML5 html.
	
_____________________

### **Attributes**

Thymeleaf is an attribute based template engine, it processes attributes and their values to build it's DOM tree.

* `th:text`: this attribute is responsible for displaying text that is evaluated from the expression inside it, it will process the expression and then display the text **html-encoded**, 
Example:
`<p th:text="#{home.welcome}">Welcome to our grocery store!</p>`

* `th:utext`: Similar to previous attribute but this one display text **unescaped** for more inforamtion check [using_texts](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#using-texts)
* `th:attr` : Takes an HTML attribute and sets it's value dynamically, example: '
`<input type="submit" value="Subscribe me!" th:attr="value=#{subscribe.submit}"/>`
The `value` attribute will be set to the value of `#{subscribe.submit}` after processing, replacing the supplied `value="Subscribe me!"`
* `th:value`,`th:action`,`th:href, th:onclick`...etc: Those attributes can be used as a shorthand of the `th:attr` syntax as equally equivilant to it, so the attribute `th:action` is equal to `th:attr="action="` 
* `th:attrappend`: This will not replace the attribute value, but will only append the value to it, example: `th:attrappend="class=${' ' + cssStyle}"`, for more information check [setting_attribute_values](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#setting-attribute-values)
* `th:each`: This is the iteration attribute, it is analogous to Java's for-each loop: `for(Object o : list)`, but its syntax is 

	```html
	<tr th:each="prod,iterStat : ${prods}" th:class="${iterStat.odd}? 'odd'">
	     <td th:text="${prod.name}">Onions</td>
	     <td th:text="${prod.price}">2.41</td>
	    <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
	</tr>
	``` 
	
	The `th:each="prod,iterStat : ${prods}"` is equivilat to `for(Product prod : prods)` and the `iterStat` is the status variable of the iteration, it contains inforamtion about current iteration like its number,index,total count ...etc. 
The iteration object `prod` can then be accessed in the context of the tag `<th>`, meaning it will only exist within the tag that it's been defined in, for more information check [iteration](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#iteration)
* `th:if`: Evaluates the conditions specified in the attribute and if they are true, the tag is displayed, if not they are not displayed, example : `th:if="${user.admin}"`
* `th:unless`: Is the opposite of `th:if`, it will display the tag if the value is false, so `th:unless="${user.admin}"` is equal to `th:if="${!(user.admin)}"`
* `th:switch` and `th:case`: Those attributes are used to create a swtich statement, `th:switch` will hold the variable to switch on, and `th:case` will evaluate the case statements for this variable, example

	```html
	<div th:switch="${user.role}">
		<p th:case="'admin'">User is an administrator</p>
		<p th:case="#{roles.manager}">User is a manager</p>
		<p th:case="*">User is some other thing</p>
	</div>
	```
	
	for more information check [conditional_evaluation](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html#conditional-evaluation)

__________

### **Expressions**
Thymeleaf works based on many expressions, thymeleaf has different expression syntax other than the traditional `${variablename.propertyname}` syntax, namely:

* `#{message.in.proprties.file}` similar to the **i18n** resolver in **JSF**, this expressions will look for the value provided in the localization properties files provided to the application.
Example: `<p th:text="#{brand.name}">Brand Name</p>`, when using spring it will use the `MessageSource` of spring
* `${variable}`: This is the variables expression, if your expression should evaluate to a variable or you have a variable in your `model` as an attribute, you must use this expression to access it, other expressions are used for different purposes and may not function correctly with variables, example:
`<span th:text="${today}">13 february 2011</span>`
* Thymeleaf provides some predefined variables that can be accessed using the `${#variableName}` syntaxt and they are:
	1. `#ctx` : the context object.
	2. `#vars`: the context variables .
	3. `#locale` : the context locale.
	4. `#httpServletRequest` : (only in Web Contexts ) the         				`HttpServletRequest` object.
	5. `#httpSession`: The session object of current session
	6. `#dates` : utility methods for `java.util.Date` objects : formatting , component extraction, etc.
	7. `#calendars` : analog ous to #dates , but for `java.util.Calendar` objects .
	8. `#numbers` : utility methods for formatting numeric objects .
	9. `#strings` : utility methods for String objects : contains , startsWith, prepending /appending , etc.
	10. `#objects` : utility methods for objects in general.
	11. `#bools` : utility methods for boolean evaluation.
	12. `#arrays` : utility methods for arrays .
	13. `#lists` : utility methods for lists .
	14. `#sets` : utility methods for sets .

	Example:

          <span th:text="${#locale.country}">
	and 
	
          <span th:text="${#calendars.format(today,'dd MMMM yyyy')}">13 May 2011</span>
	  
* `*{property}`: This is used the same way as the `${variable}` but works on selected objects, i.e. objects which are set using `th:object` attribute, for example

```html
<div th:object="${session.user}">
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```

This will access properties on `${session.user}` object directly using the `*{...}` syntax, like for `*{firstName}`, this is equal to using `${session.user.firstName}`
*Note*: The `th:object` is defined only in the context of the tag it's declare on, this means that it's not available outside the context of that tag.
* `@{/link/path}`: This will create a link to the path specified relative to the deployment context, so if the application is deployed at context **my-app**, then the generated path will be **/my-app/link/path**.
To add get parameters use `@{/link/path(param=value)}` which will generate **/link/path?param=value**
For Path variables use: `@{/link/{pathVariable}/path(pathVariable=${variable})}`
which will replace the **{pathVariable}** with the value from **${variable}**

*  Literals: You can also write some normal literals instead of any expressions, 
	* "'the literal string'": You can write normal strings between two **''**  single quotes
	*  "3 + 2": Normal numeric expressions
	* "false","true" and "null": are evaluted to normal `false`,`true` and `null` expressions
	* "singleWordToken": tokens with single words do not need single quotes and can be written as they are.
* `${#fields}`:  Spring MVC adds another predined varable which is `#fields`, it refers to `spring-form`  fields and their validation erros, mainly used for error validation
* `${@beanName.method()}`: Also spring specific, a bean method call expression, this will call a method on a spring bean called `beanName`, it will look for the bean in the current spring context and then execute this method
