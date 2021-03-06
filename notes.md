Check versions

Set up VM (basic)
  `vagrant box add bento/centos-7.2 --provider=virtualbox`
Run / launch VM
  `vagrant init bento/centos-7.2`
Spin up the VM instance - make available to remote into using Vagrant
  `vagrant up`
Login
  `vagrant ssh`
Install ChefDK on the VM (he uses 0.18.3)
  `curl https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chefdk -c stable -v 0.18.30`
Use CLI editor - (Vim)
  `sudo yum install -y vim`
Common commands
  `vagrant status`
  `vagrant ssh-config`
  `vagrant reload`
    - use to reload config after you've made any changes
  `vagrant suspend`
    - way to suspend the state of the install at end of day
    - powers down the VM; keeps the drive spinning
  `vagrant destroy --force`
    - kills VMs defined in this vagrant file

### Use AWS
Login to console
Launch instance
Go to marketplace
  - Search for Centos 7
  - Select CentOS 7 (x86_64) - with Updates HVM
    - Type:           t2.micro
    - Instance:       
      - Number:           1 (for now)
      - Rest of:          default (for now)
    - Storage:
      - Type:             General purpose SSD
      - Delete on Term:   checked
    - Tags:
      - Name:             his convention is to use "today's" date
    - Security Group:
      - Use existing:     http, https, ssh
    - Review and launch:
      - Click launch
        - Use existing key-pair AWSstuff.pem
        - Will get error about unprotected if using a new key (without performing `chmod`)
  Connect to instance:
    - Normal format using centos as user on remote
      - "ssh -i AWSstuff.pem centos@ec2- - - - - "
    - Install ChefDK on the instance
      `curl https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chefdk -c stable -v 0.18.30`
    - Set up CLI editor - (Vim)
      `sudo yum install -y vim`
  Terminate instances at end of night to reduce cost

### Chapter2
Connect to remote vagrant instance
Create (and save) recipe called "hello.rb" at root
  ```file 'hello.txt' do
  	content 'Hello, world!'
  end
  ```
  - Creates a file called hello.txt
  - Puts the content "Hello, World!" into new file
Run the (Ruby) recipe on remote
  `sudo chef-client --local-mode hello.rb`
  - local-mode: not using chef server at this time
  - Per output on screen
    - Setup
      - Starts Chef Client - (you can see version)
      - Resolves cookbooks for runlist
    - Once set up
      - Converges resources
      - Steps through the recipe (echoing out what it is doing)
    - Sends notice of what done (once done)
Configuration Management systems (purpose)
  1. Item potency
    - Knowing / realizing desired "state" reached
    - Achieved by Chef-client to apply configurations to bring system into desired state using scripts (recipes)
  2.  Discovering system facts
    - Ojai - name of Chef tool used to do
    - Detecting information about system being configured
      - generic into that can be applied to multiple systems even if different
      - e.g. network interfaces, OS details, user info
  3. Templating system
    - Embedded Ruby - Chef tool used to do
    - Provides ability to use generic information (and some logic) to produce config files, so we do not have to create custom for each server
    - Typically support variable, conditionals, loops
  4. 3rd Party extensions
    - Strong / robust community means that there is probably something available for your custom solution
      - Chef supermarket, ruby gems, knife plugins

### Chapter3
Resources (3-1,2)
  - Officially - it is a statement of configuration policy
    - Type: Means that it defines the smallest __configurable__ piece in a recipe (aka building block)
      - e.g. users, groups, files, packages, services, etc
    - Desired State: describes the desired state of an element of your infrastructure and the steps needed to bring that item to the desired state
      - e.g. should package be installed, should version be updated, should file permissions be changed?
    ___See what-is-a-resource.pdf___
      - Examples:
        - package (install httpd)
          ```package 'httpd' do
              action :install
            end
          ```
        - service (_enable_ start ntp on reboot and _start_ now )
          ```service 'ntp' do
              action [ :enable,  :start ]
            end
          ```
        - file1 (create file and fill with content)
          ```file '/etc/motd' do
              content 'This computer is the property of...'
            end
          ```
        - file2 (delete specific file)
          ```file '/etc/php.ini.default' do
              action :delete
            end
          ```
        - Pay attention to the last 2 slides
          - Format / breakdown of recipe
          - Note: that there is a 'default' _Action_ for each type
  Convergence (3-3)
    ___See Test-and-Repair.pdf___
  Lab (3-4)
    - Add packages to a recipe (called setup.rb)
      - tree      
      - ntp       
    - manage the /etc/motd file
      - content = "This server is the property of _your name_"
      - permissions = owner and group are "root"
    - What I figure it _(setup.rb)_ should look like
      ```# ~/setup.rb

        package 'tree' do
          action :install   #/default, so not expressly needed
        end

        package 'ntp'

        file '/etc/motd' do
          content 'This computer is the property of Thomas Jones'
          action :create    #/default, so not expressly needed
          owner 'root'
          group 'root'
        end
      ```
    - Run on local host using `sudo chef-client --local-mode setup.rb`
    - Check using
      - `which tree`
      - `which ntp`
      - `cat /etc/motd`
      - Running recipe again does nothing (new)
  Organizing resources w/ recipes
    - Ruby is synchronous
      - top-to-bottom (grouping)
        - package
        - file
        - service
        - etc
      - left-to-right (sequence)
        - enable
        - start
        - etc

### Cookbooks
https://docs.chef.io/cookbooks.html
___See cookbooks-overview.pdf___
  - Create cookbooks
    - Include / hold recipes
    - Note the parts (files) / structure
      - Spec / test not covered here, but important (big picture for each respective role)
Git (4-4)
___See Revision-with-Git.txt___
  - Went further to update the metadata.rb (version) to reflect the change(s) made per above txt.
    - `vim metadata.rb`
      - changed version to `0.2.0`
        - note: symantic versioning (major.minor.patch)
      - `:wq`
    - `git add .``
    - `git commit -m "version 0.2.0 release - added support for emacs to setup.rb"`
Lab
  - Create cookbook called apache
    - `chef generate cookbook cookbooks/apache`
  - Create server.rb recipe inside apache folder
    - `chef generate recipe cookbooks/apache/ server`
      - .rb extension not needed if using chef generate
      - auto-populates your spec / test folders with server.rb
      - use of "generate" also auto-populates structure with a default.rb
        - use for orchestration - call other recipes (see later)
  - Open `vi cookbooks/apache/recipes/server.rb`
    - Install apache
    - Create / edit file index.html (personalize to confirm)
    - Start the service (apache) installed earlier
    ``` package 'httpd' do
          action :install
        end

        file '/var/www/html/index.html' do
          content '<title>This is my first cookbook website.</title>   <h1>Hello World!!</h1>'
        end

        service 'httpd' do
          action [ :enable,  :start ]
        end
      ```  
  - Check syntax - `chef exec ruby -c cookbooks/apache/recipes/server.rb`
  - Run the recipe - `sudo chef-client -z cookbooks/apache/recipes/server.rb`
  - Check the site - `curl localhost`

### Convergence
See which cookbooks you have: `ls cookbooks/`
See the structure of cookbooks/apache: `tree cookbooks/apache/`
Using `--runlist` or `-r`  option to specify cookbook and/or recipe
  - General: `sudo chef-client -z --runlist "cookbook::recipe"`
  - Specific: `sudo chef-client -z --runlist "apache::server"`
  - Used (working along): `sudo chef-client -z -r "workstation::setup"`
Running multiple recipes
  - `sudo chef-client -z -r "recipe[apache::server],recipe[workstation::setup]"`
    - Syntax: _recipe [cookbook::recipename]_
    - Comma to separate, __but no space after__
    - Can combine `-zr` if desired

### Calling a recipe from other recipes
Use the default.rb (mentioned earlier) to wrap cookbooks / hijack cookbooks
  - Open `vi cookbooks/workstation/recipes/default.rb`
  - Add dependency
    - General `include_recipe "cookbook::recipe"`
    - Specific `include_recipe "workstation::setup"`
    - Save file `:wq`
      - Essentially have default.rb call setup.rb
      - Limited at this time (with this syntax) to recipes in "workstation" cookbook
  - Run the default.rb `sudo chef-client -z -r "recipe[workstation]"`
      - Specifying "default" is not necessary
LAB - have cookbooks/apache/recipes/default.rb call server.rb
  - Open `vi cookbooks/apache/recipes/default.rb`
  - Add dependency `include_recipe "apache::server"`
    - Save file `:wq`
  - Run the default.rb `sudo chef-client -z -r "recipe[apache]"`

### System Inventory / Discovery tool - Ohai
- The Node Object - you may want / need to know information about the systems you are deploying (or to).
  - Examples:
    - Name:       `hostname`
    - IP Add:     `hostname -I`
    - CPU speed:  `cat /proc/cpuinfo`
    - Memory:     `cat /proc/meminfo`
  - You could hardcode it into the motd _content_ in workstation/recipes/setup.rb and rerun chef-client, but limits you
- Ohai
  - Command line: `ohai`
    - Required by chef-client (part of Chef DK)
    - Gathers / returns a grip of info in json format
      - so you can get value for requested keys
        - `ohai ipaddress`
        - `ohai hostname`
        - `ohai memory`
      - or drill down to subkeys using `/` notation
        - `ohai memory/total`
        - `ohai cpu/0/mhz`
    - Automatically runs every time chef-client executed
      - Any time client runs to bring system into desired state, an _ohai_ gets generated (rebuilt) automatically
      - Shoves this info into "the Node Object", which represents the host's specific details
        - The values are called _node attributes_
        - We can query and request info back (rather than hardcode in a recipe)
  - Discussion of _string interpolation_ to help understand how to do
    - Initial thought / work
      - Open `vi cookbooks/workstation/recipes/setup.rb`
      - Return single attribute `node['ipaddress']`
      - Return subkey attributes `node['memory']['total']`
    - How to stringify and inject (in Ruby)?
      1. Declare a variable (attribute)
      2. Address '' vs " "
        - Wrap the whole string in "" to let it know that something complicated is coming.
        - puts is Ruby version of _print_
      3. Escape the string to deal with a variable by setting it off with `#{ }` ; which is normal Ruby
    - Example
    ```# print the statement 'I have 4 apples'
        apple_count = 4
        puts "I have #{apple_count} apples"
    ```
    - Real - incorporate into motd in setup.rb
      - Wrap the entire _content_ with " "
      - Insert the attributes wrapped in #{ }
    ```file '/etc/motd' do
        content "This server is the property of Thomas Jones
        HOSTNAME: #{node['hostname']}
        IPADDRESS: #{node['ipaddress']}
        CPU: #{node['cpu']['0']['mhz']}
        MEMORY: #{node['memory']['total']}
        "
    ```
  - Run the client `sudo chef-client -zr "recipe[workstation]"`
  - Look at contents of motd file `cat /etc/motd`
    __notice that the queried values are output, not the _code (or syntax)___
      HOSTNAME: localhost
      IPADDRESS: 10.0.2.15
      CPU: 1991.999
      MEMORY: 1016860kB
LAB - update the index.html file to contain Hostname and IPADDRESS
- Open file `vi c/server.rb`
- Update content
```package 'httpd' do
    action :install
  end

  file '/var/www/html/index.html' do
    content "<title>This is my first cookbook website.</title>   
    <h1>Hello World!!</h1>
    <h2>HOSTNAME: #{node['hostname']} </h2>
    <h2> IPADDRSS: #{node['ipaddress']} </h2>"
  end

  service 'httpd' do
    action [ :enable,  :start ]
  end
```
- Run the client `sudo chef-client -zr "recipe[apache]"`
- Update change management: `vi cookbooks/apache/metadata.rb`
  - Update version to '0.2.0' - minor change, no patch
  - git
    - `git status`
    - `git add .`
    - `git commit -m "index.html include attributes for ipaddress and hostname"`

### Using template resources
Templates - embedded Ruby files (_erb_)
Why? - ___See exploring-templates.pdf___
Build
  - Add template to cookbooks/workstation/ for the motd
    - `chef generate template cookbooks/workstation/ motd`
  - Check the structure `tree cookbooks/workstation/`
  - Cut over to ___See template-file-and-ERB.pdf___
    - ERB definition (and _tags_) starting on 7-7
      - `<% ...%>` run code inside brackets, but do not print (or insert) results to template file
      - `<%=...%>` Angry squid means evaluate code in brackets, then print directly to output (template) file
  - Open template `vi cookbooks/workstation/templates/motd.erb`
    - Copy _"content"_ of motd to motd.erb instead
    - Edit to reflect
      1. No longer part of a string
      2. I want the values printed (so use proper tag)
      ```This server is the property of Thomas Jones
        HOSTNAME: <%= node['hostname'] %>
        IPADDRESS: <%= node['ipaddress'] %>
        CPU: <%= node['cpu']['0']['mhz'] %>
        MEMORY: <%= node['memory']['total'] %>
      ```
      3. save file `:wq`
Update file that will invoke the template
  - Open `vi cookbooks/workstation/recipe/setup.rb`
  - Edit - change file to point at template, instead
    - Still want to "create"
    - Use "source" instead of "file"
      - Do not have to specify absolute path __as long as template is where it is supposed to be__ (_in the templates folder_)
    ```template '/etc/motd' do
        source 'motd.erb'
        action :create
      end
    ```
    - Save file `:wq`
  - Run `sudo chef-client -zr "recipe[workstation]"`
    - Verify that the "what" has not really changed, just the "how" `cat /etc/motd`
- Update change management: `vi cookbooks/workstation/metadata.rb`
  - Update version to '0.2.1' - minor change, and a patch
    - No real update to functionality, rather refactor to make more readable
  - git
    - `git status`
    - `git add .`
    - `git commit -m "refactored motd file with a template"`
- Passing in "variables" to be evaluated (rather than strictly Ruby)
  ___See https://docs.chef.io/resource_template.html ____ for more template documentation
  - Example
    - Open `vi cookbooks/workstation/recipe/setup.rb`
      - Edit - add variables
      - Note ` => ` for what variable resolves to
      ```template '/etc/motd' do
          source 'motd.erb'
          variables(
            :name => 'Tomas'
            )
          action :create
        end
      ```
      - Save `:wq`
    - Open `vi cookbooks/workstation/templates/motd.erb`
      - Edit to reference the variable declared above
      - Note ` @ ` for reference within the erb tags
      ```This server is the property of Thomas Jones
        HOSTNAME: <%= node['hostname'] %>
        IPADDRESS: <%= node['ipaddress'] %>
        CPU: <%= node['cpu']['0']['mhz'] %>
        MEMORY: <%= node['memory']['total'] %>
        NAME:
      ```
    - Run `sudo chef-client -zr "recipe[workstation]"`
    - Verify `cat /etc/motd`
LAB - use template for index, rather than file
- Create `chef generate template apache/ index.html`
- Open `vi cookbooks/apache/recipes/server.rb`
  - Copy file material
    - Paste to `vi apache//templates/index.html.erb`
      - change from string interpolation to erb tags
    - Build real (proper) HTML file, if you want
    ```<html>
        <head>
           <title>This is my first cookbook website.</title>
        </head>
        <body>
            <h1>Hello World!!</h1> <br>
            <h2>HOSTNAME: <%= node['hostname'] %> </h2> <br>
            <h2> IPADDRSS: <%= node['ipaddress'] %> </h2>
        </body>
      </html>
    ```
    - Save `:wq`
- Update `vi cookbooks/apache/recipes/server.rb'
  ```package 'httpd' do
      action :install
    end

    template '/var/www/html/index.html' do
      source 'index.html.erb'
      action :create
    end

    service 'httpd' do
      action [ :enable,  :start ]
    end
  ```
- Run `sudo chef-client -zr "recipe[apahe]"`
- Confirm `curl localhost`
- Update change management: `vi cookbooks/apache/metadata.rb`
  - Update version to '0.2.1' - minor change, and a patch
    - No real update to functionality, rather refactor to make more readable
  - git
    - `git status`
    - `git add .`
    - `git commit -m "refactored file resource with a template for index.html"`

### Other common resources
___See each section in chapter 8____
- cookbook_file         (8.2)
  - Similar to template, but for static content (No variables or dynamic content)
  - Shipped with cookbook, like a configuration file
  - Can use the `chef generate file` option to create a file location in "tree"
- remote_file           (8.3)
  - Use when you want to download a file from a remote server
  - Rather than shipping with a cookbbok, remote_file can download from somewhere else and copy to the local node (e.g. insert a picture)
  - Requires file (available locally to the template / recipe) that you want use it.
    - recipe - to declare and download it
    - file - (index) to call / src it
  - __note__ discussion of how to edit (and reload) vagrantfile to port forward
    - default is 8080, but 9000 used in Examples
    - makes it possible to render the index.html files we have been creating
- Execute               (8.4)
  - ___See setup.rb___, note / rewatch each of the sections commented out
    - Bash - ok, but could be better
      - Example
        - create a script
        - specify the user
        - put in code to create a directory
        - set / change owner
      - Lacks conditionality (tests or item potency)
        - guards - evaluate state of the node (and run...or not)
          1. not_if - prevent condition if returns __true__
            - does the directory exist?
              - if so, do not execute the code above returns true
            - could also use Ruby to test
              - open another _do_ block
          2. only_if - allows condition only if returns __true__
      - Not platform agnostic / not all systems are going to run bash (interpreter)
    - Execute - also ok, more compatible, but could be better
      - does not specify the interpreter, so allows chef to figure out for you
      - set up similar to bash example above to create script; as well as another example showing pre-existing
- Users / groups            (8.5)
  - Creating user and group resources  
    - Use `append` to do exactly that or it will overwrite the group to only have the 1 user
  - Check ___user-and-group.txt___ to see cli history
    - See user entries  `cat /etc/passwd`
    - See group entries `cat /etc/group`
- Notifications             (8.6)
  - Notify or subscribe to know about changes
    - ___See server.rb___ for syntax for both
      - Syntax
        - notifies :action, 'resource[name]', :timer
          - before - send of notify the service to take action before the service that I am embedded in   
          - delayed - execute the resource that  I am in, then send the notification to the service
          - immediate - as advertised ... do right away
        - subscribes follows the same syntax
      - Examples
        - _notifies_ (template -> service) tells the apache service that something changed and should "restart"
        - _subscribes_ (service -> template) apache listens for change rather then template push the Notifications
  - Just changing the files is not enough
    - Requires change
    - Client runs again - marking the fact that state has changed
  - Available to all Chef resources

### Troubleshooting
Chef Development Kit
- Common commands           (9.2)
  - `chef exec` - run command in context of embedded Ruby
    - Chef installs it's own Ruby (only available locally within available context)
      - check version - `chef exec ruby -v`
      - check syntax - `chef exec ruby -c cookbooks/apache/recipes/server.rb`
  - `chef env` see environment variables
  - `chef shell-init` use ChefDK as Ruby as primary Ruby
    - `chef shell-init --help` gives you instructions how to set path
Tool chain  -                (9.3) http://docs.chef.io/release/devkit
- `chef` used for workflow
  - `chef --help` self explanatory
  - `chef generate` we will us quite a bit for building
- `rspec` (not built-in), used for unit testing
- `berks` (berkshelf) used to manage cookbook versoins and dependencies
- `knife` primary means of communication with Chef server
  - `knife client list`
Linting tools
  - `foodcritic` analyze recipe code
    - http://ww.foodcritic.io sets of rules / patterns for designing cookbooks (to be portable / reusable)
  - Rubocop - (not built-in) for checking that you are following best practices
    - `rubocop cookbooks/apache`
Debugging / testing          (9.4)
- ___See debugging recipes___ - https://docs.chef.io/debug.html
  - emty-runlist
  - set deeper log level (more verbose)
  - log resource (add messages to Chef log, and where)
  - look into event handlers
  - later, maybe look into `chef-shell`
- ___See chef styling guide___ - https://docs.chef.io/ruby
Hands-on Debugging              (9.5)
- >> points out the issues in trace each time you run `sudo chef-client -zr "recipe[apache]"`
  1. Syntax error
    - space after ": "
    - fix, run again
  2.  Undefined method
    - misspelled source as "sourced" in template reference
    - fix, run again
  3. Package error
    - http not valid yum package... "no candidate version available for http"
    - should be `httpd`
    - fix, run again
  4. File not found
    - shows the error and the declaration
    - looked (`tree`) in the "templates" folder to see templates that are actually there
      - see that there is an index.html.erb ... no index1.
    - fix, run again
  5. Failed to execute
    - unable to "enable service
      - needed to fix error in service section from "http" to "httpd"
    - fix, run again
  6. No (express) error, but
    - only one resource "enable" popping up, but should be two
    - need to use array to call multiple actions  
Test Kitchen                  (9.6)
- ___See kitchen docs___ https://docs.chef.io/kitchen.html
- ___See kitchen ci___ http://kitchen.ci
- Creates local testing VM environ using `chef generate cookbook`
  - Auto-generates a "test" structure
  - find the .kitchen.yml file in the specific cookbook folder
    - test kithen configuration file
    - used to create or deploy code to whatever
    - contains
      - driver: how (format) testing environment will be generated
        - vagrant
        - docker
        - EC2
      - _provisioners:_ what goes through and runs Chef on this installation (usually do not have to change from chef_zero)
      - _verifiers:_ goes through and executes tests
        - support several frameworks
        - has built-in support for _inspec_
          - can be used for auditing, but using here for testing
      - _platforms:_ VMs/OS (centoos, ubuntu, windows, etc)
      - _suites:_ how we want to execute runlists
        - example exhibits how it maps to your cookbook's tree structure
    - Though we will be using the pre-built kitchen.yml file, we could use `kitchen-init`to do from scratch
      - `kitchen create`
        - Vagrant steps through the process
          - Pulls in (import) base box for testing
            - Bento is Chef project
          - Creates VM and set up way to speak to it
      - `kitchen converge` run outside of VM
        - Send it the cookbook / recipe it should execute, and apply the "suites" portion of the kitchen.yml file
          - calls / runs the default.rb
            - which calls the server.rb
              - installs Chef
              - converge recipes against it and puts VM in the desired state
                - sets up web server
      - `kitchen verify` run outside of VM
        - check what you have written against the VM
        -
      - `kitchen list` gives us a list of all the "platforms" we will try to create, as well as "details" for each
      - `kitchen login` to access system you
        - `curl localhost`
          - connection refused earlier because we ran it before "converge", so no recipe applied
          - once `converge`has been run, it sends entire cookbook
            - `inspec` runs (per kitchen.yml) and shows that the tests were successful
              - ___See http://inspec.io/docs for details for writing tests___
              - actual tests are visible in "tree" as "default.test.rb"
                - _describe_ port / http
                - _describe_ command / curl
              - he makes changes, and reruns the `verify`to show errors, syntax, and pretense for assistance in writing tests
                - protocol "tcp6"
                - driver "ec2"
            - Remember that we test one cookbook at a time, so each cookbook should contain its own kitchen.yml file
InSpec              (9.7)
- Compilation:  "resource collection"
  - 
