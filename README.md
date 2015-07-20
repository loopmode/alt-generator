
# alt-generator

A set of CLI generator tools to simplify the work with boilerplate code that is tedious but often neccessary.
The initial usecase was working with the Alt javascript library, a flux implementation for react.js.
However, it should be easy to define you own helpers for any language or framework.  
  
The idea is to generate actual source files from predefined templates and user-given options.
Options are given by the user's answers to predefined questions on the commandline, much like the usage of `npm init`.

There is a generator generator that can be used to easily create new helpers for some specific needs that you might have, like creating a test, view or config file.

All configuration is based on defaults, so that sometimes it may be skipped altogether, creating a default set of output in predefined locations. 
Otherwise, when you give the answers, it tries to stay out of your way, allowing you to keep hitting the enter key and intervening only occasionally.

Options may be specified by direct input, like specifying a name, or by using options in either multiple-choice or single-exclusive manner.

	alt-generator [generator-name][target-name, generator-type]

Example using the `create` generator, using the default generator type `alt-entity`.
All examples will be using this User entity.

	>> alt-generator -new User
	Skip configuration steps and use defaults? (âœ“) no () yes
	name of the entity: User
	use generators:
		(âœ“) store
		(âœ“) actions
		(âœ“) datasource
		 ( ) component
		 ( ) test
		 ( ) docs


Benefits:
- Standardized code style via templates
- Tidy source files to start with
- Unified interface for defining variable types (flowtype vs react PropTypes)
- clean code, clean docs (see docs helper!)


## Helpers overview


### docs

Creates a new readme file in the documentation folder. The default format is markdown.

Keeping the documentation completely outside of source files improves the readability of your code.
It also applies the single-responsibility concept to a higher level by keeping a clear separation
between the technical implementation in a programming language and its description in a human language.

A documentation block will be created for all class members that are not marked with `/** @private */` according to the specified template.
A link to the created markdown file will be added to `yourDocs/readme.md` (maybe index.md?)
Example using the User entity:

	>> alt-entity -docs User
	template: (âœ“) markdown ( ) other template?
	output: docs/readme/entities/User.md 


### store

Creates a new store for an entity. When using immutable.js, all properties of type object will be created as immutable Maps, and arrays as immutable Lists respectively.

Example using the User entity:

	[store] use immutable.js? (âœ“) yes ( ) no
	[store] specify a list of property names. Use comma as separator.
	>> name, email, auth
	[store] do you want to define propTypes? (âœ“) yes ( ) no

	[store] is "name" required? ( ) no (âœ“) yes					// no was default
	[store] propType for: "name"
	>>	(âœ“) string  
	>>	( ) number
	>>	( ) bool  
	>>	( ) custom type  
	[store] "name": React.PropTypes.string.isRequired 
	>> (âœ“) ok ( ) no
	
	[store] is "email" required? (âœ“) no ( ) yes
	[store] propType for: "email"
	>>	(âœ“) string  
	>>	( ) number
	>>	( ) bool  
	>>	( ) custom type  
	[store] "email": React.PropTypes.string 
	>> (âœ“) ok ( ) no
	
	[store] is "auth" required? ( ) no (âœ“) yes
	[store] propType for: "auth"
	>>	( ) string  
	>>	( ) number
	>>	( ) bool  
	>>	(âœ“) custom type  
	[store] specify a custom type type for "auth": 
	>> shape({token: any.isRequired, timestamp:number.isRequired})
	[store] "auth": React.PropTypes.shape({"token": React.PropTypes.any.isRequired, "timestamp": React.PropTypes.number.isRequired}) 
	(âœ“) ok ( ) no
	
	[store] target file:
	>> /src/app/stores/UserStore.js 		// the value was prefilled

	[store] store configuration done!


### actions

Creates a file containing action definitions.

When a Store was detected, handler methods can be created in the Store class. The handler methods will be bound to the actions using ES7 decorators.

When pairs of names are found that share the same base except of a past-tense suffix, the present-tense names are considered API request methods and no handler methods are created for them by default.

Example using the User entity:

	[actions] specify a list of actions. Use comma as separator.
	>> load, loaded, create, created, update, updated, delete, deleted, error
	
	[actions] do you want to define params? (âœ“) yes ( ) no
	
	[actions] params for 'load':
	>> id:number
	[actions] params for 'loaded':
	>> data:any
	[actions] params for 'create':
	>> name:string
	[actions] params for 'created':
	>> name:string, id:number
	[actions] params for 'update':
	>> id:number, data:shape({name:string,email:string})
	[actions] params for 'updated':
	>> 
	[actions] params for 'delete':
	>> id:number
	[actions] params for 'deleted':
	>> 

	[actions] {StoreName} found.
	[actions] Select actions to be handled in {StoreName}

	( )		load
	(âœ“) 	loaded
	( )		create
	(âœ“) 	created
	( )		update
	(âœ“) 	updated
	( )		delete
	(âœ“)		deleted

	[actions] actions configuration done!

### datasource

Creates a datasource that connects the entity with your RESTful API.

When actions are found, API request methods can be created. (TODO: clarify..)

API request methods in datasource classes always return a `Promise`.

When pairs of names are found that share the same base except of a past-tense suffix, the past-tense names are considered API response handlers and no request methods are created for them per default.

	[datasource] {ActionsName} found.
	[datasource] specify API request methods for these actions:
	(âœ“)		load
	( ) 	loaded
	(âœ“)		create
	( ) 	created
	(âœ“)		update
	( ) 	updated
	(âœ“)		delete
	( )		deleted

	[datasource] specify API request methods:
	load	 (âœ“) GET ( ) POST ( ) PUT ( ) PATCH ( ) DELETE
	create	 ( ) GET (âœ“) POST ( ) PUT ( ) PATCH ( ) DELETE
	update	 ( ) GET ( ) POST (âœ“) PUT (âœ“) PATCH ( ) DELETE
	delete	 ( ) GET ( ) POST ( ) PUT ( ) PATCH (âœ“) DELETE
	... // same procedure as with prop types



## Config file

You can configure tools using a `.altgenrc` file to specify default options.
When

	{
		"past-tense-suffix": "d"
		"generators": {
			"docs": {
				"output": "docs/entities/{file}",
				"format": "markdown"
			},
			"actions": {
				"output": "src/app/actions/{file}",
			},
			"stores": {
				"output": "src/app/stores/{file}",
			},
			"datasource": {
				"output": "src/app/datasources/{file}"
				"name-method-automap": {
					"load": "GET"
					"create": "POST"
					"update": "PUT, PATCH",
					"remove": "DELETE"
				}
			}
		\
	}

#### .bashr shortcuts

alias alt-entity="./node_modules/.bin/alt-entity"