
Introduction
============

The so called "Auto DJ" system is a collection of system to enable a dance party without the need for a (human) DJ or playlist. It is composed of three connected systems:

* **Mobile-enabled social media voting system.** Allows participants to request songs to be played. Their identities are accessed via this system, allowing names, and photos (and potentially other things), to be accessed by other parts of the system.
* **Playlist visualizer.** Displays information about user's votes and upcoming songs in a communal (read: massively projected) setting. 
* **Graph analyzer (Sylvester).** Trims graph from a potential > 2000 songs to approx. 20 relevant, timely, important, or otherwise "hot" songs for display on the visualizer.

The adj git repo contains code for the visualizer and graph analyzer. Code for the voting system is stored on a separate repo.

Architecture Overview
=====================

graph Namespace
---------------

This namespace contains objects related to a particle system / spring simulation. It is heavily modeled after the [Traer physics library](http://murderandcreate.com/physics/) (read: C++ port), and as such follows Traer's own coding conventions and styles. It wasn't built from the ground up for the purposes of this visualizer. If its capabilities aren't adequate, it's probably best to rewrite these objects from the ground up rather than modifying Traer's code.

adj Namespace
-------------

This namespace concerns objects related to the visualization of data, connection with other servers, and spawns a new application window.

adj::GraphNode Object
---------------------

This is the fundamental unit of the visualization. Each GraphNode contains a song (which it represents), and a graph::Particle object, linking it back to the physics simulation.

adj::Song Object
----------------

Songs objects represent songs, and should contain all information about a song, including artist, album, cover art, and some means of playing the song.

adj::User Object
----------------

Similar to Song, a user object contains information about a user, in particular their name and a photo. Additionally, each User contains a list of all Songs that the user has voted on.

adj::GraphOracle Object
-----------------------

The GraphOracle is the main interface for querying the Sylvester. It builds a representation of the current state of the graph, and gets a new representation. How the GraphOracle acts on this data hasn't been decided.

adj::Renderer Object
--------------------

Handles all things OpenGL related, and draws all necessary objects.

adj::SocialConnector Object
---------------------------

This object connects to the "social media" part of the system, currently implemented in HTTP. The exchange is fairly straightforward: 

1. Visualizer sends a request containing the timestamp of the last response it received
2. Social media server sends a response
3. Both sides disconnect

The visualizer's request is fairly basic (and still to be determined), but contains:

* **Timestamp** of the last response

The media server responds with the following:

* **Votes:** the songs that have been voted on, and who voted for them (specified by an ID)
* **People:** at the moment people only have 2 values associated with them, their ID, and their name
* **Blacklist:** IDs of people who have misbehaved or entered offensive information
* **Timestamp:** to uniquely identify the request

adj::net Namespace
------------------

This namespace contains objects related to connecting with other parts of the system over the network, typically by HTTP.

json Namespace
--------------

Files for the JsonCpp library, found [here](http://jsoncpp.sourceforge.net/).

Coding Conventions
==================

Almost all the code conforms to Google's C++ sytle guidelines, found [here](http://google-styleguide.googlecode.com/svn/trunk/cppguide.xml). Notable exceptions to Google's style include:

Exceptions are used 
-------------------

A function throws an exception if and only if it fails to do what it claims to do (presumably because it can't). When using exceptions, remember to:

* **Leak no resources**
* **Don't allow data structures to become corrupted.**

See *Effective C++* item 29 for more info.

Use std::shared_ptr for all memory allocation
---------------------------------------------

All instances of "new" should be wrapped in a std::shared_ptr. As a corollary, there shouldn't ever be instances of "delete" in the code. The one exception to this is with singletons objects. In this case, free pointers and calls to delete are allowed.

