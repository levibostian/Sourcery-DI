## [1.0.0] - 2020-06-04

## Added
- Add strong type properties to graph. 
BREAKING CHANGE: previous dependencies added to graph using `extension DI {}` will require changing to the new syntax.

## Changed 
- Remove UI imports

## [0.1.1] - 2019-12-16

## Added
- Better instructions in the README on 3rd-party dependencies, generics. 

## Changed 
- Created a macro and variable set to prevent as much copy/paste in the stencil. 

## Fixed 
- Custom defined dependencies benefit from overriding in tests. 

## [0.1.0] - 2019-12-02

## Added
- Create Sourcery stencil template for dependency injection. Stencil is used by manually downloading it to your project.