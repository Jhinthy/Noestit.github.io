---
layout: single
title: "Globalization"
permalink: /codeguidelines/globalization

sidebar:
  nav: "codeguidelines"
---

## Globalization

A lot of web applications have translations available for their users. For example, in Belgium it is common to provide Dutch/Englisch/French translations.
Using resources to store translations also prevents hardcoded `strings` in your views.

In order to externalize these translations in the form of `Resources`, we create a separate `Class Library` that store these resources and use the appropriate translations depending on the users `culture`.

For each language we add a `LANGUAGE.resx` resource file inside the resources `class library`.

**English**

<img src="./../assets/resxeng.png" alt="English resources" >

**Dutch**

<img src="./../assets/resxnl.png" alt="Nederlands resources" >

- Note that the resource name of correspondending translations must be the same.

**ResX Manager Visual Studio Extension**

With `ResXManager` you can manage all resources in one view. Updated example of our demo application.

Download
[ResXManager](https://haacked.com/archive/2011/01/06/razor-syntax-quick-reference.aspx/).

<img src="./../assets/Resxmanager.png" alt="ResXManager" >

Inside our application, we can replace all hardcoded string with these translations. Below is an example of calling resources in our `user` table.

```csharp
<h2>Users</h2>
<div>
    <table class="table">
        <tr>
            <th scope="col">@Resources.Global.Nickname</th>
            <th scope="col">@Resources.Global.Fullname</th>
            <th scope="col">@Resources.Global.Emailaddress</th>
            <th scope="col">@Resources.Global.Role</th>
            <th scope="col">@Resources.Global.Actions</th>
        </tr>
        @foreach (var user in Model)
        {
            @Html.Partial("_UserPartial", user)
        }
    </table>
</div>
```

**Data Annotations example**

```csharp
public class CreateUserViewModel
{
    [DisplayName(Resources.Name)]
    [Required(ErrorMessage=Resources.NameRequiredMessage)]
    [MaxLength(100)]
    public string Name { get; set; }
}
```

- In above example, the `errorMessage` will show the correct message, depending on user's selected culture.

The `DisplayName` attribute in above example is what will be displayed on the `View` when calling that property name.

```
@Html.LabelFor(model => model.Name)
@Html.ValidationMessageFor(model => model.Name)
```

- Above ` HTML helpers` will display the correct translation depending on browser language / culture settings.

- You can use translations for every string on your application. You can store full paragraphs if needed. Just add the reference to the resources library and you can use them in your view simply by calling `@Resources.ResourceName`
