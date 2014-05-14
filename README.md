# Easygrid

Provides a declarative way of defining Data Grids.

It works currently with jqGrid, google visualization and jQuery dataTables.

Out of the box it provides sorting, filtering, exporting and inline edit just by declaring a grid in a controller and adding a tag to your gsp. 

It also provides a powerful selection widget ( a direct replacement for drop-boxes )

[Official plugin page on Grails portal](http://grails.org/plugin/easygrid)

## Installation
Add the following plugin dependencies to your `BuildConfig.groovy`
```groovy
grails.project.dependency.resolution = {
    plugins {
        ...
        // EasyGrid plugin http://grails.org/plugin/easygrid
        compile ":easygrid:1.6.1"
        // For minimum functionality you need: jquery-ui and the export plugins.
        // Export Plugin http://grails.org/plugin/export
        compile ":export:1.5"
        // jQuery UI Plugin http://grails.org/plugin/jquery-ui
        compile ":jquery-ui:1.10.3"
        // For google visualization you also need google-visualization
        // Google Visualization API Plugin http://grails.org/plugin/google-visualization
        compile ":google-visualization:0.7"
        ...
    }
}
```
Check latest published version on [Grails plugin portal](http://grails.org/plugin/easygrid).

After installation you need to run `grails easygrid-setup` command


## Overview

The issues that Easygrid tackles are:

* steep learning curve for most ajax Grid framework
* once integrated into a grails project the business logic for each ajax Grid resides in multiple places (Controller, gsp). Usually, in the controller, there's a different method for each aspect ( search, export, security, etc)
* a lot of concerns are addressed programmatically, instead of declaratively (like search, formats)
* duplicated code (javascript, gsp, controllers). Each project has to create individual mechanisms to address it.
* combo-boxes are suitable only when the dataset is small. Easygrid proposes a custom widget based on the same mechanism of defining grids, where for selecting an element you open a grid (with pagination & filtering) in a dialog and select the desired element

Easygrid solves these problems by proposing a solution based on declarations & conventions.

### Demo
- [Easygrid example project](https://github.com/tudor-malene/Easygrid_example)
- [Pet Clinic - Easygrid style](http://199.231.186.169:8080/petclinic/)
- [Basic demo ](http://199.231.186.169:8080/easygrid)


### Features:

- custom builder - for defining the grid
- very easy to setup:  able to generate a Grid from a domain class without any custom configuration
- reloads the grid(s) when the source code is changed
- convenient default values for grid & column properties
- DRY: predefined column types - ( sets of properties )
- define the column formatters in one place
- customizable html/javascript grid templates
- built-in support for exporting to different formats ( using the exporter plugin )
- easy inline editing ( when using jqgrid )
- configurable dynamic filtering form -
- possibility to define master-slave grids or subgrids ( see the petclinic example )
- easy to mock and test

- Jquery-ui widget and custom tag for a powerful selection widget featuring a jquery autocomplete textbox and a selection dialog built with Easygrid ( with filtering, sorting,etc)


Concepts
--------------------

The entire grid logic is defined in the Controller (or in an outside file) using the provided custom builder. ( some particular view aspects can be defined in the gsp , using the provided taglibs)

For each grid you can configure the following aspects:

- datasource ( by default -if none specified - it uses the gorm datasource)
- grid implementation ( by default is jqGrid - can be overwritten )
- columns:
    - name
    - value ( could be a property of the datasource row, or a closure )
    - formatting ( see section below)
    - Optional specific grid implementation properties ( that will be available in the renderer)
    - filterClosure - in case the grid supports per column filtering - this closure will act as a filter ( will depend on the underlying datasource ). (for the gorm datasource - a default filterClosure is generated for each column).
- security
- global formatting of values
- export
- custom attributes

The plugin provides a clean separation between the model and the view ( datasource & rendering )
Currently there are implementations for 3 datasources ( _GORM_ , _LIST_ - represents a list of objects stored in the session (or anywhere), _CUSTOM_ )
and 4 grid implementations ( _JqGrid_, _GoogleVisualization_, _Datatables_, _static_ (the one generated by scaffolding) )

## Usage

All grids will be defined in controllers - which must be annotated with @Easygrid. 

In each annotated controller ( @Easygrid ) you can define a static closure called "grids" where you define the grids which will be made available by this controller.
Starting with version 1.4.1, the preferred way of defining grids is by using plain closures ending with the 'Grid' suffix ( similar to flows in Spring WebFlow )

The plugin provides a custom Builder for making the configuration very straight forward.

Ex:  (from the  petclinic sample)
```groovy
def ownersGrid = {
    domainClass Owner
    columns {
        id {
            type 'id'
            enableFilter false
        }
        firstName
        lastName
        address
        city
        telephone
        nrPets {
            enableFilter false
            value { owner ->
                owner.pets.size()
            }
            jqgrid {
                sortable false
            }
        }
    }
}
```

In the gsp, you can render this grid via the following tag:
```gsp
<grid:grid name="owners" addUrl="${g.createLink(controller: 'owner', action: 'add')}">
    <grid:set caption="Owners" width="800"/>
    <grid:set col="id" formatter="f:customShowFormat" />
    <grid:set col="nrPets" width="60" />
</grid:grid>
<grid:exportButton name="owners"/>
```

You also have to add the resource modules for the grid implementation:
```gsp
<r:require modules="easygrid-jqgrid-dev,export"/>
```

From this simple example, you can see how we can define almost all aspects of a grid in a couple of lines.
(datasource, rendering, export properties, filtering)

Custom UI properties can be added to the Config file ( as defaults or types), in the Controller, or in the gsp page.

The simple grid from this example - which can be viewed [here](http://199.231.186.169:8080/petclinic/overview), comes with all features - including basic default filtering. 

#### Grid Implementations:

*    **jqgrid**
        - implements [jqgrid](http://www.trirand.com/blog/)

*	**visualization**
        - implements [google visualization datatable](https://developers.google.com/chart/interactive/docs/reference#DataTable)

*	**datatables**
        - implements [datatables](http://datatables.net)

*	**classic**
        - implements the classic grails grid ( the static one generated by scaffolding )


 To create your own implementation you need to:
    1. create a service
    2. create a renderer
    3. declare the service and renderer in `Config.goovy`

On installation of the plugin , the renderer templates for the default implementations are copied to `/grails-app/views/templates/easygrid` from the plugin, to encourage the developer to customize them.


#### Grid Datasource Types:
1.	**gorm**
    * the datasource is actually a Criteria Builder created from the specified domainClass and initialCriteria. (basically it does: DomainClass.createCriteria().buildCriteria(initialCriteria) and afterwards applies the globalFilterClosure and the filterClosure from each column)
    * _domainClass_ - the domain ( mandatory)
    * _initialCriteria_  - used for complex queries
    * _globalFilterClosure_  - a filter that will be applied all the time ( the only parameter is 'params' )

    For fast mock-up , grids with this type can be defined without columns, in which case these will be generated at runtime from the domain properties (dynamic scaffolding).

    A filter closure is defined per column and will have to be a closure which will be used by a GORM CriteriaBuilder
    Starting from version 1.5.0 this datasource works only with Hibernate!

2.	**list**
    * used when you have a list of custom objects (for ex. stored in the session ) that must be displayed in the grid
    * _context_ - the developer must declare the context where to find the list ( defaults to session )
    * _attributeName_ - and the attribute name (in the specified context)
    * pagination will be handled by the framework

    The filterClosure takes 2 parameters: the Filter object and the actual row - and returns true if the row matches the filter

3.	**custom**
    * when the list to be displayed is dynamic ( generated by a closure )
    * _dataProvider_  - closure that returns the actual data ( must implement pagination )
    * _dataCount_ - closure that returns the number of items



If you want to create your own datasource (skip this as a beginner):

1. Create a service - which must implement the following methods:
      * generateDynamicColumns() - optional - if you want to be able to generate columns
      * verifyGridConstraints(gridConfig)  - verify if the config is setup properly for this datasource impl
      * list(Map listParams = [:], filters = null) - (     * returns the list of rows
                                                           * by default will return all elements
                                                           * @param listParams - ( like  rowOffset maxRows sort order)
                                                           * @param filters - array of filters )
      * countRows(filters = null)

      * in case you want to support inline editing you need to define 3 closures ( updateRow , saveRow ,delRow)
2. declare this service in Config.groovy , under easygrid.dataSourceImplementations




#### Columns section [see] (https://github.com/tudor-malene/Easygrid/blob/master/src/groovy/org/grails/plugin/easygrid/ColumnConfig.groovy)

The _name_ of each column will be the actual name of the closure. Beside the actual column name, from the _name_ property other properties can be inferred, like:
    * the label ( the column header ) can be automatically generated ( see below)
    * in case there is no property or value setting ( see below ), _name_ will be used as the column property ( see below)
    * also you can access the columns using this _name_ ( in case you want to override some properties in the taglib - see below)
    * _name_ is also used as the name of the http parameter when filtering or sorting on a column

1. Column Label:
The _label_ can be defined , but in case it's missing it will be composed automatically using the 'labelFormat' template - defined in Config.groovy. ( see comments in Config.groovy)

2. Column Value:
For each column you have to define the value that will be displayed in the cell.
There's 2 options for this:
In case the type of the grid is "gorm" or "list", and you just want do display a plain property you can use "property".

Otherwise you need to use the "value" closure, whose first parameter will be the actual row, and it will return whatever you need.

(There is a possibility to define the "property" columns more compact by using the actual property as the name of the column )

3. Javascript settings:
Another important section of each column is the javascript implementation section.
All the properties defined here will be available in the render template to be used in whatever way.

#### Filtering:

* _enableFilter_ - if this columns has filtering enabled
* _filterFieldType_ - one of the types defined per datasource - used to generate implicit filterClosures. In case of _gorm_ the type can be inferred

* _filterClosure_:
When the user filters the grid content, these closures will be applied to the dataset.
In case of grid _type_ = _gorm_, the search closure is actually a GORM CriteriaBuilder which will be passed to the list method of the domain.

#### Export
Easygrid also comes integrated with the [export plugin](http://grails.org/plugin/export)
This plugin has different settings for each export format , which can pe declared in the config file or in the grid.

Each column has an optional export section, where you can set additional properties like width, etc.

#### Column types
From the example you can also notice the _type_ property of a column.
Types are defined in Config.groovy, and represent a collection of properties that will be applied to this column, to avoid duplicate settings.

Default values:
* all columns have default values ( defined in Config)- which are overriden.


#### Formatters:
* defined globally (Config.groovy) based on the type(class) of the value ( because - usually , applications, have global settings for displaying data . Ex: date format, Bigdecimal - no of decimals, etc.)
* formatters can also be defined per column

The format to apply to a value is chosen  this way:

1. _formatter_ - provided at the column level
2. _formatName_    - a format defined in the _formats_ section of the Config file
3. the type is matched to one of the types from the _formats_ section of each datasource
4. the value as it is


####  Security:

If you define the property _securityProvider_ : then it will automatically guard all calls to the grid

Easygrid comes by default with a spring security implementation.
Using this default implementation you can specify which _roles_ are allowed to view or inline edit the grid


#### Other grid properties:

You can customize every aspect of the grid - because everything is a property and can be overriden. You can also add any property to any section of the configuration, and access it from the customizable template ( or from a custom service, for advanced use cases )


### Selection widget

The Selection widget is meant to replace drop down boxes ( select ) on forms where users have to select something from a medium or large dataset.
It is composed from a jquery autocomplete textbox ( which has a closure attached on the server side) and from a selection dialog whith a full grid (with filtering & sorting), where the user can find what he's looking for, in case he can't find it using the fast autocomplete option.
It can also by constrained by other elements from the same page or by statical values. ( for ex: in the demo , if you only want to select from british authors)

You can use any grid as a selection widget by configuring an "autocomplete" section ( currently works only with JqGrid & Gorm )

[Online demo ](http://199.231.186.169:8080/easygrid/book/create)

Like this:
```groovy
autocomplete {
    idProp 'id'                             // the id property
    labelValue { val, params ->             // the label can be a property or a closure ( more advanced use cases )
        "${val.name} (${val.nationality})"
    }
    textBoxFilterClosure {         // the closure called when a user inputs a text in the autocomplete input
        ilike('name', "%${params.term}%")
    }
    constraintsFilterClosure { params ->    // the closure that will handle constraints defined in the taglib ( see example)
        if (params.nationality) {
            eq('nationality', params.nationality)
        }
    }
}
```

* _idProp_       - the name of the property of the id of the selected element (optionKey - in the replaced select tag)
* _labelProp_    - each widget will display a label once an item has been selected - which could be a property of the object
* _labelValue_    - or a custom value (the equivalent of "optionValue" in a select tag)
* _textBoxFilterClosure_   - this closure is similar to _filterClosure_ - and is called by the jquery autocomplete widget to filter
* _constraintsFilterClosure_ - in case additional constraints are defined this closure is applied on top of all other closures to restrict the dataset
* _maxRows_ - the maximum rows to be shown in the jquery autocomplete widget


### Taglib:

Easygrid provies the following tags:

*  ``` <grid:grid name="grid_name">  ``` will render the taglib - see [doc] (https://github.com/tudor-malene/Easygrid/blob/master/grails-app/taglib/org/codehaus/groovy/grails/plugins/easygrid/EasygridTagLib.groovy)

*  ``` <grid:exportButton name="grid_name">  ``` the export button - see [doc] (https://github.com/tudor-malene/Easygrid/blob/master/grails-app/taglib/org/codehaus/groovy/grails/plugins/easygrid/EasygridTagLib.groovy)
    has all the attributes of the export tag from the export plugin, plus the name of the grid

*  ``` <grid:selection >  ```  renders a powerful replacement for the standard combo-box see the taglib document    ( see doc and Example)


### Testing:

- in each annotated controller, for each grid defined in "grids" , the plugin injects multiple methods:
       * ``` def ${gridName}Html ()```
       * ``` def ${gridName}Rows ()```
       * ``` def ${gridName}Export ()```
       * ``` def ${gridName}InlineEdit ()```
       * ``` def ${gridName}AutocompleteResult ()```
       * ```def ${gridName}SelectionLabel ()  ```



### Guide on extending the default functionality:

- on installing the plugin it will copy 4 templates ( one for each renderer) in your /templates/easygrid folder 
- You are encouraged to customize the templates so that the layout matches your needs

- also, in the configuratin folder there will be a file : EasygridConfig 

- The GridConfig object ( which defines one grid ) is designed so that you can add new properties to the grid or to any column. These properties that you add, can be accessed in the renderer templates to customize the view.
  The default behavior is that all the properties added in the jqgrid {} section ( or visualization, etc, ) will be automatically added to the created javascript object ( only use properties available in the library documentation )
  


## FAQ:

### I want to implement my first grid. What are the steps?    ###
A: First you need to annotate a controller with @Easygrid, and define the desired grid ( def gridNameGrid = {..}  )
In the gsp (if it belongs to that controller), all you have to do is:
  1) add <r:require modules="easygrid-jqgrid-dev,export"/> ( or whatever impelementation you're using )
  2) <grid:grid name="gridName"/>

### I need to customize the grid template. What are the properties of the gridConfig variable from the various templates? ###
A: Check out [GridConfig](https://github.com/tudor-malene/Easygrid/blob/master/src/groovy/org/grails/plugin/easygrid/GridConfig.groovy)

### What is the role of the Filter parameter passed to  the filterClosures?  ###
A: Check out [Filter](https://github.com/tudor-malene/Easygrid/blob/master/src/groovy/org/grails/plugin/easygrid/Filter.groovy) . 

### Why does the filterClosure of the _list_ implementation have 2 parameters? ###
A: Because on this implementation you also get the current row so that you can apply the filter on it, as opposed to the _gorm_ implementation where the filter closure is a gorm criteria.

### Isn't it bad practice to put view stuff in the controller?  ###
A: You don't have to put view stuff in the controller. You are encouraged to define column types and as many default view properties as possible in the config file. Also , you can override or set any view property in the grid tag in the gsp.

### I need to pass other view attributes to the ajax grid.  ###
A: No problem, everything is extensible, just put it in the builder, and you can access it in the template. 

### I don't use spring security, can I remove the default implementation?  ###
A: Yes you can. If there is no securityProvider defined, then no security restrictions are in place.

### Is it possible to reference the same grid in multimple gsp pages, but with slight differences?   ###
A: Yes, you can override the defined grid properties from the taglib. Check out the taglib section.

### I don't like the default export.    ###
A: No problem, you can replace the export service with your own.

### Are the grid configs thread safe?    ###
A: Yes.

### The labelFormat property is weird.  ###
A: The labelFormat is transformed into a SimpleTemplateEngine instance and populated at runtime with the gridConfig and the prefix.

### Are there any security holes I should be aware of?  ###
A: All public methods are guarded by the security provider you defined either in the config or in the grid. You can also apply your own security filter on top of the generated controller actions.

### What is the difference between Easygrid and the JqGrid plugin?   ###
A: The JqGrid plugin is a thin wrapper over JqGrid. It provides the resources and a nice taglib. But you still have to code yourself all the server side logic.
  Easygrid does much more than that.

### I use the jqgrid plugin. How difficult is it to switch to easygrid?   ###
A: If you already use jqgrid, then you probably have the grid logic split between the controller and the view.
   If you have inline editing enabled, then you probably have at least 2 methods in the controller.
   Basically, you need to strip the grid to the minimum properties ( the columns and additional properties) , translate that intro the easygrid builder and just use the simple easygrid taglib.
   If, after converting a couple of grids, you realize there's common patterns, you are encouraged to set default values and define column types, to minimize code duplication.
   After the work is done, you will realize the grid code is down to 10%.

### I have one grid with very different view requirements from the rest. What should I do.      ###
A: You can create a gsp template just for it and set it in the builder.

### The value formatting is complicated.       ###
A: It's designed to be flexible.

### Can I just replace a select box with a selection widget?    ###
A:  Yes, it's pretty easy, it's justa matter of replacing  the select tag . Nothing needs to be changed in the controller

### I want to customize the selection widget.   ###
A: Just create a new autocomplete renderer template and use the selection jquery ui widget

### I need more information on how to.. ?   ###
### I have a suggestion.  ###
A: You can raise a github ticket , drop me an email to: tudor.malene at gmail.com, or use the grails mailing list


## Version History

### 1.6.1
    Improvements:
       - small improvements on inline editing.

### 1.6.0
    Improvements:
       - inline editing was improved, so that it is able to display messages on each invalid field
       - added a new property: _listClass_ , used for the 'list' datasource

    Bugs:
       - various bugs related to inline editing & list datasource

### 1.5.2
    Bugs:
       - the sortClosure for the 'list' datasource will receive 3 arguments ( first one will be the sort order )
       - the list datasource is able to access a nested property from a context

### 1.5.1
    Improvements:
       - added some default values to the list datasource ( by default the filterDataType is String )

    Bugs:
       - added a countDistinct property to the GridConfig ( and also a smart default for handling count & the distinct projection )
       - fixed a filtering bug for jqgrid
       - fixed a stylesheet bug for datatables


### 1.5.0
    Improvements:
       - enhanced the filtering capabilities
           - default filtering for nested properties like: 'pet.owner'
           - filtering with operators ( equals, contains, greater than )
       - enhanced the sorting capabilities
           - sorting nested properties
           - customize sorting with closures
       - switched from DetachedCriteria to Criteria
           - ability to use projections
       - new taglibs for customizing the view
       - use grails Databinding for filter parameter conversion
       - ability to use nested view parameters like
            jqgrid{
               searchoptions{
                 sopt (['eq'])
               }
            }
       - cleaned up grid implementation renderers
       - Jqgrid support
            - support for subgrids
            - support for advanced searching ( thanks Ken Doig )
            - support for toolbar search with operators
            - more easy to configure searchoptions, editoptions, etc


### 1.4.6
    Bugs:
    - fixed https://github.com/tudor-malene/Easygrid/issues/44
    - fixed small jqgrid multiple sort

    Improvements:
     - moved the securityProvider to EasygridConfig

### 1.4.5
    Improvements:
     - Added default column types for float, double, BigDecimal, long and int
     - Added support for multiSort ( sorting on multiple columns )

### 1.4.4
    Improvements:
     - Added installation script: easygrid-setup. Thanks [sbglasius](https://github.com/sbglasius)

    Bugs:
     - fixed security bug on inline editing
     - added hack to fix incompatibility between grails and the export plugin ( the 'format' parameter clashes )

### 1.4.3
    Improvements:
     - Added grid initialization lifecycle closures

    Bugs:
     - the 'scaffolding' dependency is not exported any more


### 1.4.2
    Improvements:
     - Cleaned up the configuration. The defaults are in the DefaultEasygridConfig.groovy class. The custom project settings will be in /grails-app/conf/EasygridConfig
     - in development mode you can define a grid directly in the gsp ( works just for gorm ).
     - the custom inline edit closures ( updateRowClosure, etc ) will be passed the gridConfig object
     
    Bugs:
     - fixed  ListDatasourceService 
     - fixed externalGrids startup issue  on older grails versions
     - the gorm datasource also works with non Long ids


### 1.4.1
    - added possibility to declare grids in normal Controller closures - ending with "Grid". - see (https://github.com/tudor-malene/Easygrid_example/blob/master/grails-app/controllers/example/AuthorController.groovy)
    - removed the dynamic-controller dependency  ( to fix a couple of bugs )
    - removed mvel dependency


### 1.4.0
    - improved export ( support for additional filtering closure, and for custom export values -per column )
    - removed the AST transformation that was injecting the grid methods in the controller and used the dynamic-controller plugin for this
     ( this opens up the possibility of defining grids at runtime - in the next versions )
    - added first draft of a dynamic filter form definition ( it may be subject to change in the next versions )
    - added possibilty to define grids in other files
    - the autocomplete widget can be configured to display the label in the textbox ( instead of in the adjacent div )
    - changed the delegate of all the closures defined in the grid to the instance of the parent controller - so you can use any injected service or params, request, etc
    - improved error reporting on inline editing
    - upgraded jqgrid version to 4.5.4
    - cleaned up tests
    - improved performance
    - fixed bugs


### 1.3.0
    - upgraded jqgrid version to 4.4.4
    - added master-slave feature for jqgrid grids
    - support for fixed columns for datatables
    - added 'globalFilterClosure' - a filter closure that can be used independently of any column
    - upgraded rendering templates
    - improved performance
    - fixed bugs

### 1.2.0
    - refactored exporting so that to take full advantage over the export plugin

### 1.1.0
    - upgraded to grails 2.2.0
    - upgraded jqgrid & visualization javascript libraries
    - added support for default ( implicit ) filter Closures
    - added support for 'where queries' when defining initial criterias
    - added default values for the autocomplete section
    - the selection widget is now customizable
    - improved documentation


#### 1.0.0
    - changed the column definition ( instead of the label, the column will be defined by the name)
    - resolved some name inconsistencies ( @Easygrid instead of @EasyGrid, etc )
    - grids are customizable from the taglib
    - columns are accessible by index and by name
    - rezolved some taglib inconsistencies
    - replaced log4j with sl4j


#### 0.9.9
    - first version


## Upgrade

#### Upgrading to 1.6.1
 Merge _jqGridRenderer.gsp and/or _dataTablesGridRenderer.gsp

#### Upgrading to 1.6.0
 This version will break only custom inline edit closures.
 To upgrade you need to add a second parameter to saveRowClosure, updateRowClosure or delRowClosure. This parameter will be of type InlineResponse. In order to send messages to the UI you will need to use this object instead of the return values.
 Checkout the reference implementation: GormDatasourceService.updateRow


#### Upgrading to 1.5.0
 This is a major update and it will break existing grids. Please let me know ASAP if you have problems upgrading
 First of all check the [petclinic example:](https://github.com/tudor-malene/grails-petclinic) and the [example](https://github.com/tudor-malene/Easygrid_example) repositories.

- the renderers were rewritten . Any change should be ported to default properties.
- there is no longer necessary to define filterClosures in most of the cases. Easygrid generates them for you. Check the petclinic example for some examples.
- taglibs: there are nicer ways of customizing the view in 1.5.0 using grid:set - check the example


#### Upgrading to 1.4.2
- isolate all the changes you did to the easygrid section of the Config file and move them to /conf/EasygridConfig.groovy
- after that remove the entire easygrid section from Config
- if you use the custom inline edit closures ( updateRowClosure, etc ) - the fist parameter of the closure will be the actual gridConfig object 


#### Upgrading to 1.4.0
 - merge the rendering templates to benefit from the latest features
 in Config.groovy :
 - add a default.export.maxRows value
 - add default.autocomplete.autocompleteService = org.grails.plugin.easygrid.AutocompleteService
 - add default.idColName = 'id'
 - if you want to use the dynamic filtering form , you need to add the config for this
     filterForm {
         defaults{
             filterFormService = org.grails.plugin.easygrid.FilterFormService
             filterFormTemplate =  '/templates/filterFormRenderer'
         }
     }
  - replace filter.column with filter.filterable where the default search closures are defines


#### Upgrading to 1.3.0
 - merge the rendering templates to benefit from the latest features
 - some new default configs in [Config.groovy](https://github.com/tudor-malene/Easygrid/blob/master/grails-app/conf/Config.groovy)
 - the autocomplete 'constraintsFilterClosure' has only the 'params' parameter instead of filter


#### Upgrading to 1.2.0
 - in the 'defaults' section of the configuration, you must add a export section, where you need to define the service and default parameters for each export type


#### Upgrading to 1.1.0
 - on install, templates are copied to the /templates/easygrid folder ( & the default configuration was updated too )
 - filter closures now have 1 parameter which is a [Filter class](https://github.com/tudor-malene/Easygrid/blob/master/src/groovy/org/grails/plugin/easygrid/Filter.groovy)
 - the labelFormat has a slightly different format (for compatibility reasons with groovy 2.0 - see the comments )
 - 'domain' datasource has been replaced with 'gorm' - for consistency
 - _maxRows_ - has been added to the autocomplete settings
 - a _filters_ section has been added to each datasource implementation , with predefined closures for different column types


#### Upgrading to 1.0.0

- change the annotation to @Easygrid
- change the columns ( the column name/property in the head now instead of label)
- replace datatable to dataTables
- overwrite or merge the renderers

- In Config.groovy
    - the labelFormat is now a plain string:  labelFormat = '${labelPrefix}.${column.name}.label'
    - replace EasyGridExportService with EasygridExportService
    - replace DatatableGridService with DataTablesGridService and datatableGridRenderer with dataTablesGridRenderer

- configure the label format for grids
- in the taglib - replace id with name



## License

Permission to use, copy, modify, and distribute this software for any
purpose with or without fee is hereby granted, provided that the above
copyright notice and this permission notice appear in all copies.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.
