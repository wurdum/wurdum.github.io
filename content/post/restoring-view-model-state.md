+++
date = "2016-02-23T23:25:06+02:00"
title = "Restoring view model state"
categories = ["mvc4"]
+++

Today I want to share with you a pattern which I use in classic ASP.NET MVC web sites. It solves the problem of filling model* properties that won't be sent with http request (eg select items). Let's take a closer look.

Most of the model properties could be divided into two groups:

* Domain properties that characterize model
* Additional properties that describes domain properties

``` c#
// Some model
public class Account
{
    // Domain property
    public int CountryId { get; set; }
   
    // Additional property
    public IEnumerable<KeyValuePair<int, string>> Countries { get; set; }
}
```

Domain properties do not require extra logic, they come with http response and we send them within http request. Additional properties, on the contrary, cause a problem because they can't be sent along with domain properties. Additional properties are not the part of the domain model and could be large. They do not come to application with request and so these properties are empty. As result we can't validate domain properties using additional properties (like to check if selected Country is one of available Countries). When model has additional properties, they have to be filled during processing form request.
<!--more-->
So, every time we create a model, we need to fill additional properties. Simple solution is to reside filling logic in controller. But doing so we receive a code duplication - we have filling logic where we render view and we have it in action that handles form request. Clearly makes sense to extract filling logic into separate class, whose responsibility would be to populate additional properties.

``` c#
public class AccountFiller : IViewModelFiller<Account>
{
    public void Fill(Account model) {
        model.Countries = Countries.GetAvailable();
    }
}
```

As previously, we're going to fill model in two places - in controller, when we create model, and in model binder, when we bind form values to model. In model binder we fill model before validation. So we can use additional properties during validation (using FluentValidation).

``` c#
// ...somewhere in controller
public  ActionResult Account() {
    var account = new Account { CountryId = 1 };

    _accountFiller.Fill(account);

    return View(account);
}

public class AccountModelBinder : DefaultModelBinder
{
    private readonly IViewModelFiller<Account> _filler;

    public AccountModelBinder(IViewModelFiller<Account> filler) {
        _filler = filler;
    }

    protected override void OnModelUpdated(ControllerContext controllerContext, 
        ModelBindingContext bindingContext) {
        var model = (Account)bindingContext.Model;

        _filler.Fill(model);
       
        base.OnModelUpdated(controllerContext, bindingContext);
    }
}
```

This pattern works better when DI and FluentValidation are being used. It reduces code duplication and helps to build less coupled, easy to extend testable architecture of request handling pipeline.

*Hereinafter, when I say model, I actually mean view model. The question of applying view models is beyond the scope of this article and won't be discussed here.