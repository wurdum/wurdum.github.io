---
layout: post
title: "Model validation basics in ASP.NET MVC"
date: 2015-11-15 21:47:50 +0200
comments: true
categories: mvc4
---

Any system that receives user input must validate it. ASP.NET MVC applications are not an exception, that is why framework creators implemented several approaches how it could be done. In this article I'm not going into implementation details. Here I want to analyse each of approaches and choose the one which most flexible. Also, I'm not going to touch on the client validation, because I'm sure that it have to be implemented independently from server side. Well, lets look at each of approaches:

## Controller validation

Placing validation into controller is the easiest and most direct approach.

``` c#
public class User
{
    public string Login { get; set; }
    public string Password { get; set; }
}

[HttpPost]
public ActionResult CreateUser(User user) {
    if (string.IsNullOrEmpty(user.Login))
        ModelState.AddModelError("Login", "Please specify Login");

    if (string.IsNullOrEmpty(user.Password))
        ModelState.AddModelError("Password", "Please specify Password");

    if (!ModelState.IsValid)
        return View(user);

    _userService.CreateUser(user);

    return RedirectToAction("UsersList", "Home");
}

```

We verify model, add errors to ModelState and then just check ModelState.IsValid. This way has obvious advantage - simplicity. We don't have to isolate logic, any dependencies accessible to controller are also accessible to validator. Logic is easy to find and easy to read. Disadvantages are not less-obvious. Necessity to validate the same model in multiple controllers brings to [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle violation. Inability to isolate validation logic leads to troubles in testing. When system complexity arises, disadvantages outweigh the advantages. I could hardly imagine production system where usage of controller validation would be reasonable.
<!-- more -->
## Data Annotations attributes

Validation using ValidationAttribute is based on the idea that each model property has its own set of invariants which can be described by simple logic. Each attribute implements particular logic that checks invariant and can be applied to multiple properties across the application. It's a neat solution which removes duplication and puts logic in one place.

``` c#
public class MustBeStrong : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext) {
        var password = (string)value;
        var isStrong = password != null && password.Length > 6;

        return isStrong
            ? ValidationResult.Success
            : new ValidationResult("Please specify strong password", new[] { validationContext.MemberName });
    }
}

public class User
{
    [Required]
    public string Login { get; set; }
    [Required]
    [MustBeStrong]
    public string Password { get; set; }
}

[HttpPost]
public ActionResult CreateUser(User user) {
    if (!ModelState.IsValid)
        return View(user);

    _userService.CreateUser(user);

    return RedirectToAction("CreateUser", "Home");
}

```

This approach is perfectly suitable when we talk about validation of a single property. But it can't help you when model validation is based on  combination of properties. Also it is not obvious how to inject services/repositories into validation attribute.

## IValidatableObject interface

Complex validation can be done by implementing in model IValidatableObject interface. This interface contains only one method, that will be called during validation.

``` c#
public class User : IValidatableObject
{
    public string Login { get; set; }
    public string Password { get; set; }
    public string Confirm { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext) {
        if (string.IsNullOrEmpty(Login))
            yield return new ValidationResult("Please specify Login", new[] { "Login" });

        if (string.IsNullOrEmpty(Password))
            yield return new ValidationResult("Please specify Password", new[] { "Password" });

        if (Confirm != Password)
            yield return new ValidationResult("Confirm must be equal to Password", new[] { "Confirm" });
    }
}

```

Approach is similar to controller validation except the validation logic is isolated in model. At first glance it's a good way. However, it also has a significant flaw. When we talk about models in context of ASP.NET MVC, we actually mean [DTO](https://en.wikipedia.org/wiki/Data_transfer_object). And when we add validation to DTO, we violate single responsibility principle. It will remind itself when we need to inject dependencies into validation logic of bloated model.

## Custom ModelValidatorProvider

In order to fix previous problem we have to create separate class whose responsibility would be validation. Such approach can be implemented using ModelValidatorProvider. It is a factory that creates validators using model type. ASP.NET MVC allows to create custom factory that returns our custom validators.

``` c#
public class User
{
    public string Login { get; set; }
    public string Password { get; set; }
    public string Confirm { get; set; }
}

public class ProjectModelValidatorProvider : ModelValidatorProvider
{
    public override IEnumerable<ModelValidator> GetValidators(ModelMetadata metadata, ControllerContext context) {
        if (metadata.ModelType == typeof(User))
            yield return new UserModelValidator(metadata, context);
    }
}

public class UserModelValidator : ModelValidator
{
    public UserModelValidator(ModelMetadata metadata, ControllerContext controllerContext)
        : base(metadata, controllerContext) { }

    public override IEnumerable<ModelValidationResult> Validate(object container) {
        var model = (User)Metadata.Model;
        if (string.IsNullOrEmpty(model.Login))
            yield return new ModelValidationResult { MemberName = "Login", Message = "Please specify Login" };

        if (string.IsNullOrEmpty(model.Password))
            yield return new ModelValidationResult { MemberName = "Password", Message = "Please specify Password" };

        if (model.Confirm != model.Password)
            yield return new ModelValidationResult { MemberName = "Confirm", Message = "Confirm must be equal to Password" };
    }
}

// register your Provider
ModelValidatorProviders.Providers.Add(new ProjectModelValidatorProvider());

```

Custom ModelsValidatorProvider has no mentioned flaws. Validation logic is isolated in separate classes which are easy to test. Since we're implementing custom provider, we can set up validators resolving through IOC container that already exists in project. It will solve any troubles with third party dependencies access.

## FluentValidation

Each time you decide to write code to solve well known problem do a simple search and try to find a good solution that already exists. There is no wonder that library which provides the right solution to this problem is already written. Its name is a [FluentValidation](https://github.com/JeremySkinner/FluentValidation).

FluentValidation is a library that created to solve any validation tasks and contains a bunch of methods for most of simple cases. It also has an extension points that allows to adapt it for custom needs. FluentValidation has its own implementation of ModelsValidatorProvider which helps to integrate it to any ASP.NET MVC project. So there is no need to implement ModelValidatorProvider by yourself.

At this point I want to finish this brief tour of the validation approaches. Do validation right and keep code clean!