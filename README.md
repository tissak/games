CoreData relationship sample
=============================

This sample demonstrate how to use CoreData in RubyMotion without using XCode and XCode data models:

- relationships and attributes are supported
- default, optional, transient and indexed properties are supported for attributes
- optional, transient, indexed, ordered, min, max and delete-rule properties are supported for relationships
- attributes and relationships are specified declaratively and on a per object basis
- store.rb is mostly independent from the objects
- some CoreData helper/extension classes are provided in lib/
- both table views with and without sections are demonstrated


Creating a Game class
=======================

A Game is identified by a name, occurs within a given year and has a unique timestamp. All attributes are mandatory, not transient and not indexed:

	class Game < NSManagedObject
	  @sortKeys = ['timestamp']
	  @sectionKey = 'year'
	  @searchKey = 'name'
	
	  @attributes = [
		{:name => 'name', :type => NSStringAttributeType, :default => '', :optional => false, :transient => false, :indexed => false},
		{:name => 'timestamp', :type => NSDateAttributeType, :default => nil, :optional => false, :transient => false, :indexed => false},
		{:name => 'year', :type => NSInteger16AttributeType, :default => 0, :optional => false, :transient => false, :indexed => false},
	  ]

A Game has many players:

	class Game < NSManagedObject
	  @relationships = [
		{:name => 'players', :destination => 'Player', :inverse => 'game', :json => 'players', :optional => true, :transient => false, :indexed => false, :ordered => true, :min => 0, :max => NSIntegerMax, :del => NSCascadeDeleteRule},
	  ]

In addition, a Game is indexed by year (for example in a table view), is sorted by timestamp and searching is performed by name :
	
	class Game < NSManagedObject
	  @sortKeys = ['timestamp']
	  @sectionKey = 'year'
	  @searchKey = 'name'
	
Creating a Store
=================

A Store is created simply by indicating the name of the NSManagedObject classes; in our example Game and Player:

	class Store
	  DB = 'store.sqlite'
	  ManagedObjectClasses = [Game, Player]


Accessing a Game
================

The fetchedResultsController for a game is accessed as follows. You need to first reset the cache for the fetchedResultsController:

	Game.reset
	Game.controller

If you want instead to filter objects using a given search string, use:

	Game.reset 
	Game.searchController('name1')

Here's how to access the section name, the number of objects in the section and an object at the given indexpath:

	Game.controller.sections[section].name
	Game.controller.sections[section].numberOfObjects
	Game.controller.objectAtIndexPath(indexPath).name

Editing a Game
==============

A game can be edited easily; a dedicated UITableView will be created automatically to let the user edit the game's fields:

    @gameController ||= UITableViewControllerForNSManagedObject.alloc.initWithStyle(UITableViewStyleGrouped)
    @navGameController ||= UINavigationControllerDoneCancel.withRootViewController(@gameController, target:self, done:'doneEditing', cancel:'cancelEditing')
    
    @gameController.object = game
    @gameController.is_update = true
    navigationController.presentModalViewController(@navGameController, animated:true)

From Game to JSON (and vice-versa)
==================================

A Game can then easily be output in JSON:

	game.to_json

A Game can as easily be from a JSON string:

    json = '{
        "timestamp": "5/11/2012",
        "year": 2012,
		"name": "game1"
	}'
    
    Game.add do |game|
      game.from_json(json)
    end
