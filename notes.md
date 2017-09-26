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
      - Resolves cookbooks for run list
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
      - also auto-populates your spec / test folders with server.rb
  - Edit cookbooks/apache/recipes/server.rb
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
  - Check syntax - `chef exe ruby -c cookbooks/apache/recipes/server.rb`
  - Run the recipe - `sudo chef-client cookbooks/apache/recipes/server.rb`
  - Check the site - `curl localhost`



Thomas R. Jones
(720) 837-2338
ThomasRJones211@gmail.com
www.linkedin.com/in/thomasrjones211
