---
layout: post
title: How to setup an Angular 2 project with Typescript support.
---

There are two ways to create an Angular 2 project.

One is to use ES6, the other is to use Typescript. Typescript offers strong types and a cleaner way to configure
Angular 2 components called `decorators`.

Personally I prefer using typescript. It ticks all the right boxes for me, being from a Java backend. And using
`decorators` for configuring the angular metadata is similar to what I've done with Java annotations.

So how do we set up an Angular 2 project with TypeScript support?

First we will need [npm](https://www.npmjs.com/), which can be installed by following the guidelines from the
[docs](https://docs.npmjs.com/getting-started/installing-node).

#### Setting up typescript support

First we need to install typescript. With `npm` this is quite easy, we just need to type the following command.
```
    npm install -g typescript
```

Now let's create a directory named `angular2_starter`, this is going to be where all the actions going to be.

Let's switch into the directory and create the `tsconfig.json` file. This file let's us configure the typescript
compiler.

```
tsc --init --target es5 --sourceMap --experimentalDecorators --emitDecoratorMetadata
```

A `tsconfig.json` should be create in the directory now. It should look something like this.

```json
{
    "compilerOptions": {
        "target": "es5",
        "sourceMap": true,
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true,
        "module": "commonjs",
        "noImplicitAny": false
    },
    "exclude": [
        "node_modules"
    ]
}
```

Few important configurations to note are

 * The `target` option which tells the typescript compiler to compile typescript `.ts` files into ES5 version of javascript
 * The `sourceMap` options, which tells the compiler to keep the mapping files between `.ts` sources and the generated
   `.js` files.
 * The experimentalDecorators and emitDecoratorMetadata enables the typescript decorators, we will be using to configure
   Angular2
 * The `module` option, which let's us decide which way we want our code to be packaged. In this case
   [commonjs](http://wiki.commonjs.org/wiki/CommonJS)

We need to add two additional options to the `tsconfig.json` file now.

1. `outDir` This tells the compiler to output the compiled files to a specific directory.
2.  `rootDir` This tells the compiler the rood directory of our app, it will then only look at files form this directory.

```json
{
    "compilerOptions": {
        "target": "es5",
        "sourceMap": true,
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true,
        "module": "commonjs",
        "noImplicitAny": false,
        "outDir": "build",
        "rootDir": "app"
    },
    "exclude": [
        "node_modules"
    ]
}
```

Now that we have declared we are using the `app` directory as the root for our typescript files, we need to create it
inside the `angular2_starter` directory.

More compiler options can be found from the
[official documentation](http://www.typescriptlang.org/docs/handbook/compiler-options.html)

Now we need to start the typescript compiler and tell it to start watching for `.ts` file changes. If you have an IDE,
it will probably do this automatically. I'm using intelliJ WebStorm and it does this automatically.

Otherwise you just need to type in the following command form the directory your `tsconfig.json` file is located.

```tsc --watch```

You will see a message similar to this `Compilation complete. Watching for file changes.`.

Keep in mind that you shouldn't kill this process, keep it running to continually compile the typescript files.

#### Setting up Angular 2

First we need to tell `npm` to initialize our app.

```
npm init
```

You can just press enter to all the questions without any problems, unless you want to change any of them, in which
case you should.

This will create a file name `package.json`. This is where the dependencies for our app is going to be in.

It will look like this

```json
    {
      "name": "angular2_starter",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "author": "",
      "license": "ISC"
    }
```

Now to add angular as a dependency into our application using `npm`.

Type in

```
 npm install angular2 --save
```

This will install angular2, add it and it's dependencies into our `package.json` file, which should by now look like this.

```json
{
  "name": "angular2_starter",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "angular2": "^2.0.0-beta.16",
    "es6-shim": "^0.35.0",
    "reflect-metadata": "^0.1.2",
    "rxjs": "^5.0.0-beta.2",
    "zone.js": "^0.6.12"
  }
}
```

And that's it. Angular is ready to be used in our app.

Let's create a few components and take it for a test ride.

Create a file named `hello_app.ts` like the example below. Remember to keep the typescript compiler running!

```typescript
import {Component} from "angular2/core";

@Component({
    selector: "hello-app",
    template:`<h1>Hello from Angular 2!</h1>`
})
export class HelloApp {

}
```

The `Component` decorator is what we use to give the Angular 2 framework the metadata about the class.

The `selector`designates the tag that will represent this component. It can be used in HTML pages
like `<hello-app></hello-app>`.

The `template` option is how we can tell Angular what we want our component to render. Here we are keeping things very
simple since this is basically just a "Hello World" app.

After compilation you should see `hello_app.js` (the compiles javascript code), `hello_app.js.map`(the mapping file
between compiled javascript and source typescript) files inside the build folder (remember the folder we designated
as the `outDir`?).

Let's start our angular app, create another file named `bootstrap.ts` and enter the bootstrapping code.

```typescript
import {bootstrap} from 'angular2/platform/browser';
import {HelloApp} from './hello_app';

bootstrap(HelloApp).catch(error => console.log(err));
```

`bootstrap` is how we will be starting our Angular 2 application. Right now we are only using it to start our simple
application, however as we start using the full range of Angular 2 features the options we pass into the bootstrap
method will get more complex and numerous.

We now need to create an HTML file as the entry point in our app. Let's create our `index.html` file.

```html
<html>
    <head></head>
    <body>
    <hello-app>
        You will see the content between these tags while the angular app is loaded.
    </hello-app>
    </body>
</html>
```

While modules are in the ES6 specification, they are not supported in all the browsers in use today, and ES5 doesn't
have modules.

So how can we load out application in a modular way?

We will, in this instance, use [SystemJS](https://github.com/systemjs/systemjs). It's a dynamic module loader.

So we need to install it via `npm`

```
npm install --save systemjs
```

Now let's statically link it in our HTML file and load the modules.

```html
<html>
    <head>
        <script src="node_modules/angular2/bundles/angular2-polyfills.js"></script>
        <script src="node_modules/systemjs/dist/system.js"></script>
        <script>
        System.config({
        // This let's us import js files without specifying .js extension
        defaultJSExtensions: true,

        // Angular dependencies to get started with.
        map: {
        'angular2': 'node_modules/angular2',
        'rxjs': 'node_modules/rxjs'
        }
        });

        // Loading the bootstrap file to start our app.
        System.import('build/bootstrap');
        </script>
    </head>
    <body>
    <hello-app>
        You will see the content between these tags while the angular app is loaded.
    </hello-app>
    </body>
</html>
```

Now serve the app over any http server and you will be good to go.

We can use the `npm` `http-server` to quickly start serving the files over http.

```
npm install -g http-server
```

Now simply type
```
http-server
```
inside the `angular2-starter` directory and access the app.