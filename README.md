# DATS: A Data-Centric Mandatory-Access Control Platform


## DATS Template Language

### Template Tags
The DATS uses the [Mustache Template Language](https://mustache.github.io/) for constructing cross-folder views of data. Mustache is a logicless template language (e.g. there are no conditional statements) that can be used for constructing HTML pages. It works by expanding keyword tags present in a template page with data provided by the application. Therefore, all template pages written for the DATS platform must conform with the Mustache spec.

Keyword tags in the Mustache language are specified by double curly brackets. For example, {{this}} is a Mustache tag, as well as {{#this}}. There are a few types of tags with specicial meanings:

#### Variable Tags {{variable}}
These tags specify a single variable of data that is passed in to the Mustache template processor. For example:
	
Template:

	Hello, {{hello}}
        
With data:

	{"hello": "world"}
        
Generates:

    	Hello, world
        
#### Section Tags {{#section}} {{/section}}
These tags will render a section of data multiple times. If the key provided to the section tag contains an array of a repeated variable tag, each variable tag will be repeated on the template. For example:
	
Template:

	{{#hello}}
	    Hello, {{name}}
    {{/hello}}
        
With data:

	{
    	"hello": [
    		{ "name": "Lee"},
        	{ "name": "Roy"},
        	{ "name": "Jenkins"}   
      	]
    }
Generates:

    	Hello, Lee
        Hello, Roy
        Hello, Jenkins
        
        

### DATS Template Keywords
The DATS platform adds a few tags to the Mustache template language to assist developers in generating cross-folder views of their data.

##### DATS.results Section Tag
The DATS.results section tag should be used to specify where in the template page cross-folder data should be placed. The entire section is repeated for each JSON document returned by a per-folder container instance. All template pages must contain the DATS.results tag to generate cross-folder views of data.

Template:

	{{#DATS.results}}
		Displaying data across my folders: {{myFolderData}}
	{{/DATS.results}}
    
With JSON data returned from Folder 1:

	{ "myFolderData": "Folder 1" }

And JSON data returned from Folder 2:

	{ "myFolderData": "Folder 2" }

Generates the template:

	Displaying data across my folders: Folder 1
    Displaying data across my folders: Folder 2


##### DATS.enter and DATS.entercmd Section Tags
Because of DATS data-centric information flow control scheme, developers are unable to generate HTML anchor tag links that transition a user from an application instance running in a non-folder container to an application instance running in a per-folder container without assistance. The DATS.enter section tag will generate these enter links for the application developers. 

In addition to the DATS.enter tag, developers must also specify the command with which they will run their application within the folder. This command must specify to the application whether it is running in a non-folder or per-folder container (e.g. "./run nonfolder" vs "./run perfolder"). To pass the command, developers must use the DATS.entercmd section tag after each DATS.enter tag and pass a JSON list of command arguments to run. DATS will then use the provided command and URL endpoint to start the application and redirect the user's browser.

Template:

	{{#DATS.results}}
    	<a href={{#DATS.enter}}"/folder-view"{{/DATS.enter}}> This is a link to {{myFolderData}} </a>
        {{#DATS.entercmd}}["./run", "perfolder"]{{/DATS.entercmd}}
	{{/DATS.results}}
    
With JSON data returned from Folder 1:

	{ "myFolderData": "Folder 1" }

And JSON data returned from Folder 2:

	{ "myFolderData": "Folder 2" }

Generates the template:

	<a href="https://380c212a4eeb5e2acef7b1a5ea1a4bfb.dats.com/folder-view"> This is a link to Folder 1 </a>
    <a href="https://6718e4460ecb10d45cdd03dbafbef597.dats.com/folder-view"> This is a link to Folder 2 </a>
    
##### DATS.folder-name Variable Tag
The DATS.folder variable tag will simply embed the name of the folder that the JSON data came from into the template page. For example:

Template:

	{{#DATS.results}}
    	<a href={{#DATS.enter}}"/folder-view"{{/DATS.enter}}> {{DATS.folder-name}} with data {{myFolderData}} </a>
        {{#DATS.entercmd}}["./run", "perfolder"]{{/DATS.entercmd}}
	{{/DATS.results}}
    
With JSON data returned from Folder 1 with folder name Home:

	{ "myFolderData": "Folder 1" }

And JSON data returned from Folder 2 with folder name Work:

	{ "myFolderData": "Folder 2" }

Generates the template:

	<a href="https://380c212a4eeb5e2acef7b1a5ea1a4bfb.dats.com/folder-view"> Home with data Folder 1 </a>
    <a href="https://6718e4460ecb10d45cdd03dbafbef597.dats.com/folder-view"> Work with data Folder 2 </a>


## DATS Health: An Example
DATS Health is a MVC application written using the MEAN stack. The application allows doctors to manage doctor-patient appointments and schedule them through a calendar. We will use the application as a driving example for constructing a cross-folder view of data using the DATS Template Processor.

### Writing a Cross-Folder Controller

The following code snippet shows the application controller which handles the first cross-folder view a user interacts with. The first step of the controller is to determine which type of container the application is running in, either a non-folder container or per-folder container. 

If the application is running in a non-folder container, it must signal to the DATS platform its returning a cross-folder template. First, the application sets the "x-dats-crossfolder" response header to the command it wishes to use to run the per-folder application instance. It then returns the template page ("./views/encounter-overview.template") to the DATS proxy.

If the application is running in a per-folder container, then it has access to the user's data within the folder. The application simply queries the database for a list of all a user's encounters and returns the list as a JSON object.

![DATS Health Cross-Folder Controller](/images/DATS_Health-cross-folder-controller.png)


### Writing a Cross-Folder Template Page

The following code snippet shows the template page for the cross-folder view of a user's encounters. The DATS.results section tags outline the HTML code that is repeated for each folder (the following section shows the final rendered page). The template uses the DATS.folder-name variable tag to highlight the user's folder that each JSON data came from. 

For this example, the following JSON documents were returned:

From the Jack Johnson folder:

	{
    	"date": "10/13/2015",
        "patient": "Jack Johnson",
        "facility": "Seton",
        "description": "Patient came in with broken Ulna on 10/18/2015"
    }

From the Matt Espinosa folder:

	{
    	"date": "06/06/2016",
        "patient": "Matthew Espinosa",
        "facility": "Seton",
        "description": "Routine checkup on 06/06/2016"
    }

From the Daniel Craig folder:

	{
    	"date": "04/13/2015",
        "patient": "Daniel Craig",
        "facility": "Seton",
        "description": "Given Anti-inflamatory and scheduled a followup visit on 4/18/2015"
    }
    
![DATS Health Cross-Folder Template](/images/DATS_Health-cross-folder-template.png)


### The Produced HTML Page

This is the final HTML that is rendered in the browser after the page has been rendered.

![DATS Health Cross-Folder HTML](/images/DATS_Health-cross-folder-html.png)






