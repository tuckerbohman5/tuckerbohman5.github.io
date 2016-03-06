---
layout: post
title: My First CLI Gem
---

As part of the Learn.co cirriculum we are required to take assesments that consist of putting together what we have learned in the lessons to build a project of our choice that meets the requirements specified. My first project was to build a CLI Ruby Gem with the following specifications:

1. Package as a gem
2. Provide a CLI on gem installation.
3. CLI must provide data from an external source, whether scraped or via a public API.
4. Data provided must go at least a level deep, generally by showing the user a list of available data and then being able to drill into a specific item. 

I decided for my project to build a NBA-Drilldown application to list the teams and players so you could see the players of a team. The hardest part of this project was finding a good external source to scrape the information from. I spent a few weeks trying to use [NBA.com](http://www.nba.com/teams/?ls=iref:nba:gnav) however I eventually decided to use [ESPN.com](http://espn.go.com/nba/teams)

To start creating a gem I simply needed to use bundler: `bundle gem nba_drilldown`. This created the simple file structure needed to create my very first gem. Now it was time to get coding. I first decided how I wanted my program to function and I decided that I wanted the program to welcome the user and to list the teams. It was then time to set up the enviornment for the program.

I knew that I wanted the ability for a user to install the gem and then simply run `nba-drilldown` from the terminal and that would fire off the program. The bundler created that file in the bin directory for me and we will get to how to create an executable later in this post. In the bin/nba-drilldown file I coded the following: 

```ruby
#!/usr/bin/env ruby

require "bundler/setup"
require "nba_drilldown"

NbaDrilldown::CLI.new.call
```

The first line sets the enviornment to ruby so that the computer knows how to read the code. I then require bundler/setup (which pulls code from the bundler gem) and nba_drilldown which is the main module of the program. The last line is creating a new CLI object and executing the call method. The `call` method is standard procedure in Object-Oriented Ruby. I then prepared my `nba_drilldown` module. 

```ruby
require "pry"
require "nokogiri"
require "open-uri"

require_relative './nba_drilldown/cli'
require_relative './nba_drilldown/version'
require_relative './nba_drilldown/team'
require_relative './nba_drilldown/player'

module NbaDrilldown
  # Your code goes here...
end
```

In this file I required the gem dependencies needed for development and the two needed for scraping. It was also neccessary to require all of the relative files which would soon contain the different classes needed to create my program. Lets take a look at the cli class: 

```ruby
class NbaDrilldown::CLI

def call
  NbaDrilldown::Team.create_teams if NbaDrilldown::Team.all.empty?
  
  puts "Welcome to NBA Drilldown!"
  
  puts "Please enter the number for a team to see its players or type 31 to exit"
  NbaDrilldown::Team.list_teams
  puts "31. Exit!!!"

  input = gets.chomp.to_i
   if input == 31
    puts "Goodbye!"
  elsif (1..30).include?(input)
    team = NbaDrilldown::Team.find_team(input)
    team.create_players_from_team
    puts "The players of the #{team.name}"
     team.list_players
     continue?
  else
      puts "I am sorry please enter a valid team number"

    
  end
   
end

def continue?
  puts "Would you like to look at another team? Yes to return or No to exit."
    input = gets.chomp
    if input.downcase.capitalize == "Yes"
      call
    else
      puts "Goodbye!"
    end
  end

end
```

In this class we have two instance methods. `call` and `continue?` call is what starts the program and welcomes the user it then scrapes the teams from ESPN and creates a new team object for each one. The teams are then listed out and the program waits for input from the user. When the user types in a number it will search the teams for that specific team and then list the players of that team. After listing the players the `continue?` method is called and prompts the user if they would like to look at another team. If so it will execute the `call` method again but this time it will not create the teams again since they already exist. Or the program will exit. Lets take a look at the Team and Player classes now. 

```ruby
class NbaDrilldown::Team
  attr_accessor :name, :conference, :players, :url

  @@all = []

  def initialize
    @players = []
  end


  def self.create_teams
    doc = Nokogiri::HTML(open("http://espn.go.com/nba/teams"))
    doc.search("a.bi").each do |team| 
      new_team = NbaDrilldown::Team.new
      new_team.name = team.text
      new_team.url = team["href"]
      @@all << new_team
    end

  end

  def list_players
    puts "Name|Number|Position"
    self.players.each do |player|
      puts "#{player.name}|#{player.number}|#{player.position}"
    end
  end

  def self.find_team(input)
    team = @@all[input-1]
    team
  end

  def self.list_teams
    @@all.each_with_index do |team, index|
      puts "#{index + 1}. #{team.name}"
    end
  end

  def self.all
    @@all
  end


  def create_players_from_team
    self.url = self.format_team_url
      
      doc = Nokogiri::HTML(open(self.url))
      doc.search("td.sortcell a").each do |player|
      new_player = NbaDrilldown::Player.create_from_data(player)
      
      new_player.add_player_info
      new_player.team = self
      
      self.players << new_player
      end

  end

  def format_team_url
    split_array = self.url.split("/")
    new_array = []
    split_array.each do |text|
      if text == "_"
        text = "roster"
        new_array << text
      else 
        new_array << text
      end
    end
    new_array.pop
    new_array.insert(6,"_")
    new_array.join("/")
  end
end
```
```ruby
class NbaDrilldown::Player
  attr_accessor :name, :team, :conference, :position, :number, :url

  @@all = []

  def initialize

  end


  def self.create_from_data(player)
    
    #doc.search("td.sortcell a").each do |player|
      new_player = NbaDrilldown::Player.new
      new_player.name = player.text
      
      new_player.url = player['href']
      @@all << new_player
      new_player
       
  end 

  def add_player_info
    doc = Nokogiri::HTML(open(self.url))
    
      self.number = doc.search("ul.general-info li").first.text.match(/\d+/)
      self.position = doc.search("ul.general-info li").first.text.match(/[A-Z]+/)
        
  end

  def self.list_players
    @@all.each do |player|
      puts player.name
    end
  end

end
```

Both the Player and Team class have the responsibility to create Team and Player objects by scraping ESPN. Once I was able to get all of the code working it was time to finish my gem and publish it to [RubyGems.org](http://guides.rubygems.org/contributing/) To do this I first needed to make some changes to my `gemspec` file. 

```ruby 
Gem::Specification.new do |spec|
  spec.name          = "nba_drilldown"
  spec.version       = NbaDrilldown::VERSION
  spec.authors       = ["tuckerbohman5"]
  spec.email         = ["tucker.bohman@gmail.com"]

  spec.summary       = "A gem to learn about NBA teams and players"
  spec.description   = "A gem to learn about NBA teams and players more functionaility to come."
  spec.homepage      = "http://github.com/tuckerbohman5"
  spec.license       = "MIT"

  # Prevent pushing this gem to RubyGems.org by setting 'allowed_push_host', or
  # delete this section to allow pushing this gem to any host.
  #if spec.respond_to?(:metadata)
   # spec.metadata['allowed_push_host'] = "TODO: Set to 'http://mygemserver.com'"
  #else
    #raise "RubyGems 2.0 or newer is required to protect against public gem pushes."
  #end

  spec.files         = `git ls-files -z`.split("\x0").reject { |f| f.match(%r{^(test|spec|features)/}) }
  spec.bindir        = "bin"
  #spec.executables   = spec.files.grep(%r{^exe/}) { |f| File.basename(f) }
  spec.executables = ["nba-drilldown"]
  spec.require_paths = ["lib"]

  spec.add_development_dependency "bundler", "~> 1.10"
  spec.add_development_dependency "rake", "~> 10.0"

  spec.add_development_dependency "pry"

  spec.add_dependency "nokogiri"
end
```

You can change the name, author, summar, description, etc. to match your information. I changed the bindir to bin where my executable file was and also changed the executables to equal `nba-drilldown` so now my program can be ran simply by typing `nba-drilldown` in the command line. 

I then commited my final changes and pushed them to github. Then to package my program as a gem I ran `rake build` and then to push it to [RubyGems.org](http://guides.rubygems.org/contributing/) you simply run `gem push nba_drilldown-0.1.1.gem` in the package directory. NOTE: You will need to create an account with Ruby Gems before you can publish a gem there. 

You can install my gem from the terminal with `gem install nba_drilldown` and run it with `nba-drilldown`. I love that I was able to build this and will definitely add more functionality and features in the future. 


