# Angular Modules & Loading Strategies
        
At the time of writing this article Angular provides three main strategies for loading modules: 

- [Eager loading](#eager-loading)
- [Lazy loading](#lazy-loading)
- [Preloading](#preloading)

Let's jump straight in and we'll explore each of these in detail:

## Eager loading

Eager loading is Angular's default loading strategy. All eager routes are loaded before the application is started. This can have a detrimental impact on our application start up time (especially for larger applications).

It's important to consider user journeys and which common tasks need to be eagerly loaded to keep our applications robust and quick.

Let's begin by creating an application with some eagerly loaded components. We'll build on this later by implementing lazy and preloaded modules:

```powershell 
> ng new NgLoadingDemo
```

Navigate to the app.routing module and we'll create some routes for our future home and not found pages:

```typescript
import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";

// These will error as they don't exist yet.

const routes: Routes = [
  { path: "", component: HomePageComponent },
  { path: "not-found", component: NotFoundComponent },
  { path: "**", redirectTo: "/not-found" },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

At this point you might be wondering why we created our routes prior to our components 🤔. We can use the ```--module``` Angular CLI option to scaffold new components inside of a module.

Let's create our home and not found pages using the Angular CLI:

```powershell
> ng g c home-page --module app-routing.module
CREATE src/app/home-page/home-page.component.ts (287 bytes)
...
UPDATE src/app/app-routing.module.ts (488 bytes)
```
``` powershell
> ng g c not-found --module app-routing.module
...
```
Our app-module should now look like this:

```typescript
import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";
import { HomePageComponent } from "./home-page/home-page.component";
import { NotFoundComponent } from "./not-found/not-found.component";

const routes: Routes = [
  { path: "", component: HomePageComponent },
  { path: "not-found", component: NotFoundComponent },
  { path: "**", redirectTo: "/not-found" },
];
not - found;
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
  declarations: [HomePageComponent, NotFoundComponent],
})
export class AppRoutingModule {}
```

As we mentioned before, eager loading is Angular's default loading strategy. So our home and not found pages are eagerly loaded.

Serve the application, when it opens you should see that your home page component works. Once you've confirmed this try navigating to a route that doesn't exist. You should be redirected to the "not-found" route.

```bash
$ ng serve -o
```

If you've used Angular before there should be no major surprises here. We've simly create an application with two eagerly loaded pages.

## Lazy loading

The term "lazy loading" describes the concept of loading components and modules at runtime, as and when they're required.

At this point let's assume that our site has an optional user registration, and login system. Only a few of our visitors use these options so it might be nice to load these parts of the system as and when a user attempts to login, register.

We can encapsulate all of this functionality into a module and then lazy load the module as and when needed.

It's a good idea to break an application's parts into modules based on functionality, having each feature self-contained within a module.

This will help to keep your code neat and well structured. It also gives us the ability to load in the "user-signin" feature when a specific domain sub-directory is accessed (e.g. http://yoursite.com/user-signin/...)

Let's begin by creating a module for the user content feature:

```bash
$ ng g m user-signin --routing
CREATE src/app/user-signin/user-signin-routing.module.ts (255 bytes)
CREATE src/app/user-signin/user-signin.module.ts (301 bytes)
```

As you can see this created two files:

- the user-signin.module.ts module
- the user-signin-routing.module.ts module

These are akin to our app.module and app-routing.module files where our user-signin module exports our user signin routing module:

```typescript
import { NgModule } from "@angular/core";
import { CommonModule } from "@angular/common";

import { UserSignInRoutingModule } from "./user-signin-routing.module";
import { LoginPageComponent } from '../login-page/login-page.component';
import { RegisterPageComponent } from '../register-page/register-page.component';

@NgModule({
  declarations: [LoginPageComponent, RegisterPageComponent],

  imports: [CommonModule, UserSignInRoutingModule],
})
export class UserSignInModule { }

```

Our user-signin routing module looks like this:

```typescript
import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";

const routes: Routes = [];

@NgModule({
  
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class UserSignInRoutingModule {}
```

Let's define some routes for our components. We'll then generate our components and add them to our module simultaneously as we did before.

```typescript
import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";
const routes: Routes = [
  { path: "login", component: LoginPageComponent },
  { path: "register", component: RegisterPageComponent },
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class UserSignInRoutingModule {}
```

And we'll create page components for this module:

```bash
$ng g c login-page --module user-signin-routing.module
CREATE src/app/login-page/login-page.component.html (25 bytes)
CREATE src/app/login-page/login-page.component.spec.ts (650 bytes)
CREATE src/app/login-page/login-page.component.ts (291 bytes)
CREATE src/app/login-page/login-page.component.scss (0 bytes)
UPDATE src/app/user-signin/user-signin-routing.module.ts (379 bytes)

ng g c register-page --module user-signin/user-signin-routing.module
CREATE src/app/register-page/register-page.component.html (27 bytes)
CREATE src/app/register-page/register-page.component.spec.ts (664 bytes)
CREATE src/app/register-page/register-page.component.ts (299 bytes)
CREATE src/app/register-page/register-page.component.scss (0 bytes)
UPDATE src/app/user-signin/user-signin-routing.module.ts (480 bytes)
```

Now our user-signin-routing.module.ts should look like this:

```typescript
import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";

const routes: Routes = [
  { path: "login", component: LoginPageComponent },
  { path: "register", component: RegisterPageComponent },
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class UserSignInRoutingModule {}
```

We now need to go back to our app routing module and define a route for all of our user signin (i.e our user-signin module). As before we add a path to the routes collection however the signature is a little different this time:

``` typescript
 {
    path: "user-signin",
    loadChildren: () =>
      import("./user-signin/user-signin.module").then(
        (m) => m.UserSignInModule
      ),
  },
```

As you can see this route loads children "on the fly" by dyanamically importing the module when the route is accessed.

### Proving It Works

Don't believe me? Why should you? I wouldn't believe me either. After all, Seeing is believing.

We need a way to observe that this is working by opening browsers dev tools and clicking on the network tab and launching your site. Now navigate to /user-signin/login

![image](./images/lazy-loading-test.png)

Notice that your browser only loads in the module when navigating to the /user-signin route. 

Later we'll revist Lazy loading and we'll implement it in conjunction with route guards. To prevent modules loading when users don't have basic access.

## Preloading

In contrast to lazy loading, preloading occurs immediately after eagerly loaded components have initialised and the application is started.

Preloading components requires the use of a strategy. Angular has a built in PreloadAllModules strategy that simply pre-loads all modules defined within a router configuration.

Fine-grained control of preloading can be achieved using custom preloading strategies. This enables us to conditionally pre-load modules based on our own conditional logic.

Let's create a simple preloading strategy to see how this works: 

Modify the application routing module to include the PreloadAllModules strategy like so:

``` typescript
@NgModule({
  imports: [RouterModule.forRoot(routes, {
    preloadingStrategy: PreloadAllModules // <- Preloading
  }),
  ],
  exports: [RouterModule],
  declarations: [HomePageComponent, NotFoundComponent],
})
export class AppRoutingModule { }
```

When you serve the application and navigate home you'll see that the user signinin module we created earlier is lazily loaded although we've not navigated to the /user-signin route yet. 

This is because our component has been preloaded emmediately after the application was started.

![Image showing the module preloaded in the networking tab of our browsers dev tools](./images/pre-loading-test.png)

### Combining Lazy Loading and PreLoaded Components with Custom Strategies

Notice that in the previous example all components that were lazily loaded are now preloading. This is because we used the PreloadAllModule. What if we want to mix Preloading and Lazy loading together? As the title of this section suggests this is where custom loading strategies come in to play.

Custom loading stratagies are simply classes/services that extend the PreloadingStratedy and implement the preload method.

Let's create one now:

1. Generate a new service using the Angular Cli

``` powershell
> ng g s CustomPreloadingStrategy
CREATE src/app/custom-preloading-strategy.service.spec.ts (449 bytes)
CREATE src/app/custom-preloading-strategy.service.ts (153 bytes)
```

This will create the following 

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class CustomPreloadingStrategyService {

  constructor() { }
}

```

Extend the PreloadingStrategy class by implementing the preload function and add a call to super() in our classes constructor:

``` typescript
import { Injectable } from '@angular/core';
import { PreloadingStrategy } from '@angular/router';

@Injectable({
  providedIn: 'root'
})
export class CustomPreloadingStrategyService extends PreloadingStrategy {
  
  constructor() { 
    super();
  }
  
  preload(route: import("@angular/router").Route, fn: () => import("rxjs").Observable<any>): import("rxjs").Observable<any> {
    throw new Error("Method not implemented.");
  }
  
}
```

So now that we have the custom preloading strategy we need to give it some logic to differentiate which clases we want to preload and which we don't. 

Angular's route objects allow us to append meta data. We can use this metadata to append a flag indicating whether or not we wish to preload the module or not. 

Appending this to our user-signin route in our app-routing.module file could look like this: 

``` typescript
 {
    path: "user-signin",
    data: { preload: true },
    loadChildren: () =>
      import("./user-signin/user-signin.module").then(
        (m) => m.UserSignInModule
      ),
  },
```
The preload function itterates over the lazy loaded routes. It supplies a delgate function which returns an observable. We can trigger a preload by calling the function and returning it's result. 

We need to modify our custom routing strategy to use the preload flag we defined earlier. If the flag is present and true we will call the delegate function and return it's output. Alternatively we can just return an observable of null.

``` typescript
import { Injectable } from '@angular/core';
import { PreloadingStrategy } from '@angular/router';

@Injectable({
  providedIn: 'root'
})
export class CustomPreloadingStrategyService extends PreloadingStrategy {
  
  constructor() { 
    super();
  }
  
  preload(route: import("@angular/router").Route, fn: () => import("rxjs").Observable<any>): import("rxjs").Observable<any> {

    if (route.data && route.data["preload"] === true) {
      return fn(); // preload
    } else {
      return of(null); // return observable of null of not preloading.
    }
  }
  
}
```

Try it out, you should now see that only the routes you decorated with metadata are preloaded. 

🎉 Congratulations you now know how to implement Eager, Lazy and Preloading strategies. Next we'll revist lazy loading and see how we can use this in the context of route guards.


## Lazy Loading & Route Guards 

Imagine for a moment that we have a new requirement to have a profile page for logged in users. 

We don't want to lazily load this route until we can validate that the user has been authenticated. If the user navigates to the profile route prior to authenticating we may want to redirect them to the login page. 

Let's take a look at how we can implement this in our application. First we need a module for all guarded components. We'll then add our profile component to this newly created module. Now that we know what we're doing we can go this on a single line.

``` powershell
> ng g m auth-guarded --routing; ng g c profile --module auth-guarded/auth-guarded.module.ts

CREATE src/app/auth-guarded/auth-guarded-routing.module.ts (255 bytes)
CREATE src/app/auth-guarded/auth-guarded.module.ts (301 bytes)
CREATE src/app/profile/profile.component.html (22 bytes)
CREATE src/app/profile/profile.component.spec.ts (635 bytes)
CREATE src/app/profile/profile.component.ts (280 bytes)
CREATE src/app/profile/profile.component.scss (0 bytes)
UPDATE src/app/auth-guarded/auth-guarded.module.ts (382 bytes)
``` 

> note that I'm using powershell, if you're using bash you can simply use ```&&``` to combine your two commands.

add a route for the profile component in the auth-guarded-routing.module file as we've done before: 

``` typescript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { ProfileComponent } from '../profile/profile.component';

const routes: Routes = [
  {
    path: "profile",
    component: ProfileComponent
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class AuthGuardedRoutingModule { }

```

Then lazy load add this module to our app.routing.module as we've done for the other components: 

``` typescript

import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";
import { HomePageComponent } from "./home-page/home-page.component";
import { NotFoundComponent } from "./not-found/not-found.component";

const routes: Routes = [
  { path: "", component: HomePageComponent },
  { path: "404", component: NotFoundComponent },
  {
    path: "user-signin",
    loadChildren: () =>
      import("./user-signin/user-signin.module").then(
        (m) => m.UserSignInModule
      ),
  },

  {
    path: "auth-guarded",
    loadChildren: () =>
      import("./auth-guarded/auth-guarded.module").then(
        (m) => m.AuthGuardedModule
      ),
  },
  { path: "**", redirectTo: "/404" },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
  declarations: [HomePageComponent, NotFoundComponent],
})
export class AppRoutingModule { }

```

At this point I think our routes are looking a little bit ugly. Let's rename them to /authentication and /user. In the real world we should probably refactor the modules too but I don't think we need this to do this for the purposes of this document.

```typescript
import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";
import { HomePageComponent } from "./home-page/home-page.component";
import { NotFoundComponent } from "./not-found/not-found.component";

const routes: Routes = [
  { path: "", component: HomePageComponent },
  { path: "404", component: NotFoundComponent },
  {
    path: "authentication",
    loadChildren: () =>
      import("./user-signin/user-signin.module").then(
        (m) => m.UserSignInModule
      ),
  },

  {
    path: "user",
    loadChildren: () =>
      import("./auth-guarded/auth-guarded.module").then(
        (m) => m.AuthGuardedModule
      ),
  },
  { path: "**", redirectTo: "/404" },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
  declarations: [HomePageComponent, NotFoundComponent],
})
export class AppRoutingModule { }

```

Now we need to implement a route guard, route guards have life cycles which in turn use callback functions. These callbacks are defined in different interfaces. For the purposes of loading in the module when authenticated we need to use the CanLoad interface: 

``` powershell
> ng g g auth/auth

? Which interfaces would you like to implement? CanLoad
CREATE src/app/auth/auth.guard.spec.ts (331 bytes)
CREATE src/app/auth/auth.guard.ts (410 bytes)

```

As you can see, this has created the file ```src/app/auth/auth.guard.ts```. Let's take a look at that file now: 

``` typescript
import { Injectable } from '@angular/core';
import { CanLoad, Route, UrlSegment, ActivatedRouteSnapshot, RouterStateSnapshot, UrlTree } from '@angular/router';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanLoad {
  canLoad(
    route: Route,
    segments: UrlSegment[]): Observable<boolean> | Promise<boolean> | boolean {
    return true;
  }
}
```

As you can see we have a canLoad method where we can have some logic to determine whether or not the user is currently logged in. Typically we'd inject a service into this module and use that service provide a flag representing the authentication status. 

Let's create a mock service for this now just to prove the point: 

``` powershell
> ng g s auth/auth
CREATE src/app/auth/auth.service.spec.ts (347 bytes)
CREATE src/app/auth/auth.service.ts (133 bytes)
```

Modify the service to give it a property that represents the user's logged in state: 

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  public isAuthenticated: boolean = false;
  constructor() { }
}
```

Now we're going to modify our authentication guard to use the service and we'll also use the angular router to redirect the user to the login page if they're not currently logged in:


```typescript
import { Injectable } from '@angular/core';
import { CanLoad, Route, UrlSegment, ActivatedRouteSnapshot, RouterStateSnapshot, UrlTree, Router } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanLoad {
  constructor(private router: Router, private authservice: AuthService) { }

  canLoad(route: Route): boolean {
    if (this.authservice.isAuthenticated === false) {
      this.router.navigateByUrl("/authentication/login");
    }
    return this.authservice.isAuthenticated;
  }
}
```

Finally we need to hook up our authentication route guard inside of our app routing module like so: 

``` typescript
import { NgModule } from "@angular/core";
import { Routes, RouterModule } from "@angular/router";
import { HomePageComponent } from "./home-page/home-page.component";
import { NotFoundComponent } from "./not-found/not-found.component";
import { AuthGuard } from './auth/auth.guard';

const routes: Routes = [
  { path: "", component: HomePageComponent },
  { path: "404", component: NotFoundComponent },
  {
    path: "authentication",
    loadChildren: () =>
      import("./user-signin/user-signin.module").then(
        (m) => m.UserSignInModule
      ),
  },

  {
    path: "user",
    canLoad: [AuthGuard],
    loadChildren: () =>
      import("./auth-guarded/auth-guarded.module").then(
        (m) => m.AuthGuardedModule
      )
      ,
  },
  { path: "**", redirectTo: "/404" },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
  declarations: [HomePageComponent, NotFoundComponent],
})
export class AppRoutingModule { }
```

Nagivate to http://localhost:4200/user/profile and you'll should see that the profile module loads. 

Now try changing the property in the authentication service to false and you will be redirected to the login page.

 > 💡 There's nothing stopping us from implementing a role based system where we parts of our application will only load for specific users. If we're allowing users to remain logged in between sessions then we could event these modules to load as part of a preload strategy but that's beyond the scope of this article. 