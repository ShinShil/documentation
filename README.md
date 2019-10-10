# Intro
The main purpose of accepted project architecture is to organize code in the way when the new developers can easily extend and support applications without investigating the whole project. Also, a lot of effort has been put on making DRY cheap from terms of development time.

# Module and builds overview
The project consists of two modules: client and server, each of them can be developed and tested independently.

![image](https://user-images.githubusercontent.com/11064696/66562800-adfbf500-eb5c-11e9-986c-18b68e5ffada.png)

As builds are independent, so, possible to have issues with versioning, there are approaches resolve builds or resolve sources versions, currently it is resolved on src level by VCS, both modules lies under one VCS and build can be done only from VCS, so you never make builds with different versions.

As we have separate builds, our solution is scalable and if we need in future to we can split builds, however. For this case, we can use VCS created timestamp as a version.

## Client module

### Develop and build
Client module is consists of one react application. This is an SPA, which can be developed with webpack dev server, which is also part of the client module. Once development done developer can build the react application and put the result to the server module.

### Technologies

- react
- react-redux
- typescript
- es7
- npm
- webpack
- node

### Project structure
![image](https://user-images.githubusercontent.com/11064696/66563604-65ddd200-eb5e-11e9-8d95-efd1c8fc18cf.png)

#### jsx
JSX contains react components. All the components should be stateless, here means that component can have stated, but that state should be defined by the application state, not but component itself. They can be logically splitted on three categories

- components - reusable components, blocks, f.e. input, dropdown, sidebar. Usually, they have some inputs to setup how component will be rendered.
- layouts - reusable ways of organizing components. Usually layouts have jsx, that should be rendered in some places. 
- pages - it is the top module, where first layout is used

We need components and layout because usually, we want unified UI on the web application, it is possible only if reuse layouts and components instead of copy-paste.

#### lib
Lib is a layer for npm packages and services. We should wrap calls to external packages with this layer, son in future if we have any troubles we can easily switch the package. F.e. 
![image](https://user-images.githubusercontent.com/11064696/66566989-8d389d00-eb66-11e9-9f36-9a217a6d32a1.png)

we are exporting three functions from server-models.d.ts, if we want to get rid of Axios, it will be easy just to change it in one place.

Also, it helps to understand what code dependent on the external dependencies.

##### lib/redux
![image](https://user-images.githubusercontent.com/11064696/66567166-eb658000-eb66-11e9-8bcb-42afbaf01bd4.png)

For some libs it makes sense to provide its own structure, f.e. in redux there is a separate folder per piece of state. 

#### styles 
Shared styles

#### types
model type definitions files

#### utils
Shared functionality, that can be used across the project

### Client module conventions

- create a separate directory per component, if the component has more than one file, f.e: component, component styles, component constants
- always try to use external package, instead of writing own code. Less code we are writing - less code we have to support
- lib/axios/{controllerName}.ts has one to one mapping with the controller on the server, so in future, it can be auto-generated
- define interfaces for all web API requests
- interface names start with I
- name jsx component files in pascal case
- use function declaration on the root file instead of an anonymous function
- all jsx components should be looking in the same structure, f.e. sample for component with name History below, all parts are optional, except that default export should have the name of the component - History.
```
interface IInputs {
}

interface IOutputs {
}

interface IProps extends IInputs, IOutputs {

}

function HistoryFunction({}:IProps) {
    return ...
};

function mapStateToProps(state: IState): IInputs {
    return {};
}

function mapDispatchToProps(dispatch: any): IOutputs {
    return {};
}

const History = connect(mapStateToProps, mapDispatchToProps)(HistoryFunction);

export default History;
```
- components should stateless, if component has some state, it should managed from the application state, components state should stored in redux stored and managed only by actions. This point separates business logic from the components and provides the ability to get access to any piece of state in any piece of code and for any update function. F.e. you have sign-in button, If you need to sign in also in some other place, you can use redux action.
- DRY no duplicate code
- SOLID apply this principle when designing new modules 
- YAGNI follow YAGNI, but so that don't break SOLID and DRY. Rarely possibly to break SOLID or DRY in small pieces of code, because of time, but never on the highest. I.g. if we have c1,c2,c3 modules that should be work communicate with SOLID and DRY, but their implementation doesn't matter.

## Server module
### Overview
There are two applications: web API and web app, webapi provides api actions, like signIn, getApplicationInfo, web app just serves the SPA build from the client module.

### WebAPI Technologies
- .net core3.0
- kestrel
- mediator
- entity-framework
- ODBC MS Access Driver to get access to Product Owner access database
- dapper
- SQLite 
- no database server, there two databases that are stored as filed under the VCS: access and mdb files

### WebAPI architecture

On a high level, the code is splitted on to modules that can call each other with commands with interfaces in the mean generic contract that implement by the module.
![image](https://user-images.githubusercontent.com/11064696/66570474-f2dc5780-eb6d-11e9-9e8d-8361e8133902.png)


On-screen you can see dependencies directions between modules. F.e. both, WebAPI and Core are dependent on Utils, or in other words, Utils is used on both WebAPI and Core. It is important that only one module depends on external modules(f.e. NuGet packages), it is Core. Also, important to mention that they use each throw interface. So we have never written:
```
Class1 t = new Core.Class1();
```
instead we will use dependency injection to get it on runtime
```
IClass1 t = DI.GetClass1() , usually it will be injected with a constructor
```

![image](https://user-images.githubusercontent.com/11064696/66569050-fb7f5e80-eb6a-11e9-8a83-7f0ede566be3.png)

#### os.core 
Operations that are depends on external libraries and all related to them files. Here
- commands - mediator commands. All commands should be organized in the same way, as on sample below. 
![image](https://user-images.githubusercontent.com/11064696/66569190-51ec9d00-eb6b-11e9-8b15-55691e525bda.png)
On the screen, SignIn is the command class, and it has Command, Result and Handler classes.
- database - EF setup

#### os.utils
Operations that are shared for the project. F.e. IAppSettings can be used both in os.webapi and os.core

### Server module conventions
- Never use implementation, always use DI, preferably with constructor
- All controller actions should call mediator command, and not contain any logic
- Use `RaisedException` for an exception that you want to throw
- If you need commands for a new package to create a separate directory in os.core
- DRY no duplicate code
- SOLID apply these principles when designing new modules 
- YAGNI follow YAGNI, but so that don't break SOLID and DRY. Rarely possibly to break SOLID or DRY in small pieces of code, because of time, but never on the highest. I.g. if we have c1,c2,c3 modules that should be work communicate with SOLID and DRY, but their implementation doesn't matter.

# Architecture benefits
- Scalable
- All general tasks well defined, so it is possible to use auto-generation to speed development 
- Follows DRY and SOLID
- TIght coupled
- Use small amount of concepts
- Possible to work separately on client and server
- Possible to two split client application development on layouts and components

# Application screenshots
![image](https://user-images.githubusercontent.com/11064696/66561466-cfa7ad00-eb59-11e9-9e5c-f421f8a52e5a.png)

![image](https://user-images.githubusercontent.com/11064696/66561491-df26f600-eb59-11e9-8be4-6a3130e84718.png)

# TO DO (I am going to implement this on weekends) 
- client localization
- creating new surveys
- editing existing surveys
- creating templates for surveys
- add localization for surveys
- improve mobile devices support
- add storybook and bit
- setup unit tests for server and client module
- setup api tests
- setup e2e test for client module
- use generator to provide to generate ts models, api controllers and axios api calls
- provide default templates for server command and ts component, and redux piece


