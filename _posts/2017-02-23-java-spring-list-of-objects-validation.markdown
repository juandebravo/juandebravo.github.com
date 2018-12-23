---
layout: post
title: "Java. Validating a list of objects"
categories: [java, spring, validation, jsr-303]
---

Let's assume you have the following requirements for a simple API you're implementing:

* It should handle HTTP POST requests in the endpoint `/books`.
* The HTTP body should contain a non-empty list of objects, let's call the object *Book*.
* A *Book* object should contain three fields:
  * name: non empty string
  * publish date: valid year in gregorian calendar
  * rate: number between 1 and 5

For whatever reason, you've come up to the conclusion that implementing a REST API in Java & Spring is a good choice.
You have read that Java is a commonly used programming language and that Spring is the usual framework to be used, being [DropWizard]() another good choice.

Your environment is:
* JDK 8
* Spring
* Spring Boot

How can you validate that a HTTP request to your endpoint `/books`? Well, it's not as straight
forward as you may have been told. Here we go!

### Bean Validation in Java

JSR stands for Java Specification Request, and now we go for the spec number XYZ :sweat:

[JSR 303](https://www.jcp.org/en/jsr/detail?id=303) standarizes how, by using annotations, you can define rules for validating a Java Bean.

A very simple example:

{% highlight java %}
import javax.annotation.NotEmpty;

public class Book {

  // This annotation is defined in JSR 303 spec
  @NotEmpty
  private String name;

  // Constructors, getters, setters, oh Java...
}

{% endhighlight %}

[JSR-349](https://www.jcp.org/en/jsr/detail?id=349) introduces some improvements in JSR-303. **Which ones?**

JSR 303/349 are specifications, and JDK ? includes in the standard library the required interfaces
for the annotations to be syntactically correct in a Java program. Yet you need to use a library that
implements the annotations that the JSR defines (NotNull, NotEmpty, etc).

Spring includes the Spring Validation framework, that implements JSR 303/349. Bear in mind
that this is related to Bean validation, not Object validation. A List is an Object,
but it's not a Bean :wink: :wink:.

### Bean validation in Spring
By default, Spring uses [Hibernate Validator](http://XYZ) as JSR 303 validator.

### List validation in Spring

Find out how to validate a RESTFul request when posting to a spring controller using the bean validation API and spring's validator interface.

this is not a Spring issue but a Bean Validation JSR implementation limitation. Which is in this case Hibernate Validator.

Spring Validation implements Bean Validation(JSR 303/349) as opposed to Object validation.


The @Valid annotation is part of java bean validation API and is not a spring-specific construct.
By adding it to the input parameter within a method in @Controller we will trigger validation.
Adding the @RequestBody will indicate the annotated method parameter should be generated
from the body of the HTTP request. In other words, spring will use jackson to transform
json from the body of the request to a java object.

Spring framework has the flexibility to be configured with both JSR-303, JSR-349 bean validation or by implementing spring Validator interface to meet your organizations choice of validation.

By examining actuator endpoint http://localhost:8080/beans, OptionalValidatorFactoryBean is declared as the mvcValidator and uses hibernate validator implementation.

Spring validation's documentation: http://docs.spring.io/spring/docs/current/spring-framework-reference/html/validation.html



### Option 1
http://stackoverflow.com/questions/28150405/validation-of-a-list-of-objects-in-spring

@RequestBody @Valid ValidList<CompanyTag> categories

public class ValidList<E> implements List<E> {

    @Valid
    private List<E> list;

    public ValidList() {
        this.list = new ArrayList<E>();
    }

    public ValidList(List<E> list) {
        this.list = list;
    }

    // Bean-like methods, used by javax.validation but ignored by JSON parsing

    public List<E> getList() {
        return list;
    }

    public void setList(List<E> list) {
        this.list = list;
    }

    // List-like methods, used by JSON parsing but ignored by javax.validation

    @Override
    public int size() {
        return list.size();
    }

    @Override
    public boolean isEmpty() {
        return list.isEmpty();
    }

    // Other list methods ...
}


### Option 2

I think the most elegant solution is to create a custom Validator for Collection and a @ControllerAdvice that registers that Validator in the WebDataBinders, take a look to Spring validation for RequestBody parameters bound to collections in Controller methods (http://stackoverflow.com/questions/34011892/spring-validation-for-requestbody-parameters-bound-to-collections-in-controller/36790509#answer-36790509)

Tip: Use \@InitBinder("attrName") instead or do initBinder in specific controller.

Validator:

import java.util.Collection;

import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;

/**
 * Spring {@link Validator} that iterates over the elements of a
 * {@link Collection} and run the validation process for each of them
 * individually.
 *
 * @author DISID CORPORATION S.L. (www.disid.com)
 */
public class CollectionValidator implements Validator {

  private final Validator validator;

  public CollectionValidator(LocalValidatorFactoryBean validatorFactory) {
    this.validator = validatorFactory;
  }

  @Override
  public boolean supports(Class<?> clazz) {
    return Collection.class.isAssignableFrom(clazz);
  }

  /**
   * Validate each element inside the supplied {@link Collection}.
   *
   * The supplied errors instance is used to report the validation errors.
   *
   * @param target the collection that is to be validated
   * @param errors contextual state about the validation process
   */
  @Override
  @SuppressWarnings("rawtypes")
  public void validate(Object target, Errors errors) {
    Collection collection = (Collection) target;
    for (Object object : collection) {
      ValidationUtils.invokeValidator(validator, object, errors);
    }
  }
}

Controller advice:

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.InitBinder;

/**
 * Controller advice that adds the {@link CollectionValidator} to the
 * {@link WebDataBinder}.
 *
 * @author DISID CORPORATION S.L. (www.disid.com)
 */
@ControllerAdvice
public class ValidatorAdvice {

  @Autowired
  protected LocalValidatorFactoryBean validator;


  /**
   * Adds the {@link CollectionValidator} to the supplied
   * {@link WebDataBinder}
   *
   * @param binder web data binder.
   */
  @InitBinder
  public void initBinder(WebDataBinder binder) {
    binder.addValidators(new CollectionValidator(validator));
  }
}

Another example (https://www.leveluplunch.com/java/tutorials/017-validate-spring-rest-webservice-request/)

Bean

public class PolicyResource {

    private String name;

    //..
}

Validator

import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;

public class PolicyValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return PolicyResource.class.equals(clazz);
    }

    @Override
    public void validate(Object target, Errors e) {

        ValidationUtils.rejectIfEmpty(e, "name", "name.empty");

        PolicyResource p = (PolicyResource) target;

        //perform additional checks
        //if name already exists or ?
    }
}

Controller

@InitBinder
protected void initBinder(WebDataBinder binder) {
    binder.setValidator(new PolicyValidator());
}

http://stackoverflow.com/questions/4308224/hibernate-validation-of-collections-of-primitives

