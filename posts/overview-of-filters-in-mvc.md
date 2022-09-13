---
title: 'Overview of filters in MVC'
date: 'September 14, 2022'
excerpt: 'In ASP.NET MVC, a user request is routed to the appropriate Controller and action method. We need situations where you want to execute some logic before or after an action method executes. ASP.NET MVC provides filters for this.'
cover_image: '/images/posts/img3.jpg'
---

In ASP.NET MVC, a user request is routed to the appropriate Controller and action method. We need situations where you want to execute some logic before or after an action method executes. ASP.NET MVC provides filters for this. 

ASP.NET MVC filter is a custom class where you can write logic that needs to be executed before or after an action is called. There are mainly 4 kinds of filters in MVC.

1. Authorization Filters
2. Action Filters
3. Result Filters 
4. Exception Filters

To implement custom filters on the above filters, we can implement the required filter interface. Let us discuss one by one and implementation of custom filters in each type. 

## Authorization Filters

This filter handles authorization and authentication logic. This logic gets executed before the action gets executed. We implement the authorization of an action just by decorating the authorize attribute to an action as shown below. 

```csharp
[Authorize]    
public ActionResult Index()    
{    
    return View();    
} 
```

Now, let us learn how to customize the authorization for our application. To implement custom authorization, we extend the authorizeAttribute class to our custom class as shown below. Now, we override the authorizeCore and then implement the custom logic that determines the user is authorized or not.

In the below sample code, we have extended our custom Authorization class with the Authorize attribute and implemented the authentication as well. If the user is not authenticated, it returns false as in line 15. And it will not execute the action. In line 22, we are determining whether the logged in user is authorized for the action called. If it returns false, it prevents the action execution.


```csharp
namespace Application.Filters    
{    
    public class CustomAuthorize:AuthorizeAttribute    
    {    
        ISecurityRepos securityRepos;    
        public CustomAuthorize()    
        {    
            securityRepos = new ISecurityRepos();    
        }    
        protected override bool AuthorizeCore(HttpContextBase httpContext)    
        {    
            IPrincipal user = httpContext.User;    
            if (!user.Identity.IsAuthenticated)    
            {    
                return false;    
            }    
            string action, controller;    
            MvcHandler mvc = httpContext.CurrentHandler as MvcHandler;    
            action = mvc.RequestContext.RouteData.Values["action"].ToString();    
            controller = mvc.RequestContext.RouteData.Values["controller"].ToString();    
            string path = string.Format("{0}/{1}", controller, action);    
            bool isAuthorized = securityRepos.IsAuthorized(path,user.Identity.Name);    
            return isAuthorized ;    
        }    
    }    
}  
```

Since we have defined the logic for our custom authorization,  we just decorate the action with CustomAuthorize attribute to implement this authorization.

```csharp
[CustomAuthorize]      
public ActionResult Index()      
{      
    return View();      
}  
```

Now, that we have learned how to implement AuthorizeCore, let us learn how we can implement HandleUnauthorized requests. 

Whenever the authorizeCore method returns false, the HandleUnauthorizedRequest method is called, where we can implement the logic to handle unauthorized requests, as shown below.

```csharp
protected override void HandleUnauthorizedRequest(AuthorizationContext filterContext)    
{    
           filterContext.Controller.ViewData["AuthorizationError"] = "You are not authorized for this action";    
}
```

## Action Filters

These implement the IActionFilter interface that have onActionExecuting and OnActionExecuted methods. OnActionExecuting runs before the Action. We can use the action filters for logging the action calls. We can also check for any exception occured during action execution and log the exception details into your database. 

```csharp
public class LoggingFilterAttribute : ActionFilterAttribute    
{    
    public override void OnActionExecuting(ActionExecutingContext filterContext)    
    {    
        filterContext.HttpContext.Trace.Write("(Logging Filter)Action Executing: " +    
            filterContext.ActionDescriptor.ActionName);    
    
        base.OnActionExecuting(filterContext);    
    }    
    
    public override void OnActionExecuted(ActionExecutedContext filterContext)    
    {    
        if (filterContext.Exception != null)    
            filterContext.HttpContext.Trace.Write("(Logging Filter)Exception thrown");    
    
        base.OnActionExecuted(filterContext);    
    }    
}   
```

We shall learn the implementation of logging the exception into database using ActionFilters in my next article.

## Result Filters

These implement the IResultFilter. IResultFilter mainly has two methods - OnResultExecuting and OnResultExecuted. OnResultExecuting gets executed before the ActionResult is executed. OnResultExecuted runs after the result.

```csharp 
public override void OnResultExecuted(ResultExecutedContext filterContext)    
  {    
      Log("OnResultExecuted", filterContext.RouteData);          
  }    
    
  public override void OnResultExecuting(ResultExecutingContext filterContext)    
  {    
      Log("OnResultExecuting ", filterContext.RouteData);          
  }   
 
 ```

 Same as Action Filters, we can use Result Filters for logging the steps. In real time usage, we can use ResultFilters to log exceptions when an exception is raised when a View is executed. 

## Exception Filters

These implement IExceptionFilter and it gets executed if there is an unhandled exception thrown during the execution of code. Exception filters can be used for tasks, such as logging or displaying an error page. 

We can implement the IExceptionFilter by implementing the OnException method and implementing our custom logic within the OnException method, as shown below.

```csharp
namespace Application.Filters    
{    
    public class CustomException:FilterAttribute,IExceptionFilter    
    {    
        public void OnException(ExceptionContext filterContext)    
        {    
            if (!filterContext.ExceptionHandled && filterContext.Exception is NullReferenceException)    
            {    
                filterContext.Result = new RedirectResult("ErrorPage.html");    
                filterContext.ExceptionHandled = true;    
            }    
        }    
    
            
    }    
} 
```

As shown in the above code, we can redirect to a custom Error page whenever we get an unhandled exception.

In the next part of Filters in MVC, we shall learn order of execution of filters, different levels where we can apply these filters, and also, we will learn Action filters and Exception filters with more details.
