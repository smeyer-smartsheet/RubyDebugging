---
marp: true
theme: gaia
color: #000
colorSecondary: #333
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.jpg')
paginate: true
---
# Debugging Ruby

Welcome to the lecture on debugging Ruby with the rdbg debugger! In this session, we'll explore various techniques and tools to effectively debug Ruby applications.

---
<style scoped>section { font-size: 20px; }</style>

# Topics to be Covered

1. Interactive rdbg CLI: Introduction to the rdbg command-line interface and its capabilities.
2. Debugging with VS Code Extension: Exploring the features and benefits of the rdbg debugger extension for Visual Studio Code.
3. Using the Console to Debug: Leveraging the Rails console to interactively debug and manipulate data in your application.
4. Debugging Views in Rails: Utilizing the debug helper to inspect and debug views in your Rails application.
5. The Benchmark Gem: Introduction to benchmarking techniques using the benchmark gem to measure code performance.
6. The Benchmark-memory Gem: Understanding memory benchmarking using the benchmark-memory gem to analyze memory consumption.
7. Object#send and Common Pitfalls: Exploring the usage of Object#send and identifying common pitfalls in Ruby code.
8. Class Loading in Rails: Understanding Rails' autoload and eager_load mechanisms for loading classes.
9. Searching Code: Using regular expressions to search and find specific code patterns.
10. Q/A & Live Demos: Engaging in interactive Q/A sessions and live demonstrations.

Let's dive into the exciting world of Ruby debugging and enhance our development skills!

---

# Debugging Ruby with Rdbg

**Introduction to rdbg debugger**
*rdbg is a debugger for Ruby, which allows us to inspect our app via cli commands.*

```ruby
rdbg -e 'your_script.rb'
```

**Configuring local environment for remote debugging**

*To enable rdbg, setup alocal port forwarding via ssh.*
```bash
ssh -R 12345:localhost:12345 user@remote
```
---
<style scoped>section { font-size: 20px; }</style>

# Interactive rdbg CLI
```bash
/usr/local/bin/rdbg [options] -- [debuggee options]

Debug console mode:
    -n, --nonstop                    Do not stop at the beginning of the script.
    -e DEBUG_COMMAND                 Execute debug command at the beginning of the script.
    -x, --init-script=FILE           Execute debug command in the FILE.
        --no-rc                      Ignore ~/.rdbgrc
        --no-color                   Disable colorize
        --no-sigint-hook             Disable to trap SIGINT
    -c, --command                    Enable command mode.
                                     The first argument should be a command name in $PATH.
                                     Example: 'rdbg -c bundle exec rake test'

    -O, --open=[FRONTEND]            Start remote debugging with opening the network port.
                                     If TCP/IP options are not given, a UNIX domain socket will be used.
                                     If FRONTEND is given, prepare for the FRONTEND.
                                     Now rdbg, vscode and chrome is supported.
        --sock-path=SOCK_PATH        UNIX Domain socket path
        --port=PORT                  Listening TCP/IP port
        --host=HOST                  Listening TCP/IP host
        --cookie=COOKIE              Set a cookie for connection

  Debug console mode runs Ruby program with the debug console.

  'rdbg target.rb foo bar'                starts like 'ruby target.rb foo bar'.
  'rdbg -- -r foo -e bar'                 starts like 'ruby -r foo -e bar'.
  'rdbg -c rake test'                     starts like 'rake test'.
  'rdbg -c -- rake test -t'               starts like 'rake test -t'.
  'rdbg -c bundle exec rake test'         starts like 'bundle exec rake test'.
  'rdbg -O target.rb foo bar'             starts and accepts attaching with UNIX domain socket.
  'rdbg -O --port 1234 target.rb foo bar' starts accepts attaching with TCP/IP localhost:1234.
  'rdbg -O --port 1234 -- -r foo -e bar'  starts accepts attaching with TCP/IP localhost:1234.
  'rdbg target.rb -O chrome --port 1234'  starts and accepts connecting from Chrome Devtools with localhost:1234.

Attach mode:
    -A, --attach                     Attach to debuggee process.

  Attach mode attaches the remote debug console to the debuggee process.

  'rdbg -A'           tries to connect via UNIX domain socket.
                      If there are multiple processes are waiting for the
                      debugger connection, list possible debuggee names.
  'rdbg -A path'      tries to connect via UNIX domain socket with given path name.
  'rdbg -A port'      tries to connect to localhost:port via TCP/IP.
  'rdbg -A host port' tries to connect to host:port via TCP/IP.

Other options:
    -v                               Show version number
        --version                    Show version number and exit
    -h, --help                       Print help
        --util=NAME                  Utility mode (used by tools)
        --stop-at-load               Stop immediately when the debugging feature is loaded.
```
---
<style scoped>section { font-size: 20px; }</style>

# Examples
```bash
# Connect to a remote Ruby app (PORT: 12345):
$> rdbg -A 12345
```

```bash
$> b your_script.rb:10 # Set breakpoint on <file>:<line>
```
```bash
$> b your_script.rb:15 if: some_condition # Break if <expr> is true at specified location
```
```bash
$> b /lib\/your_script/ path: <path> # Break if the path matches to <path>
```
```bash
$> del 1 # Delete specified breakpoint
```
```bash
$> info locals # Show information about the current frame (local variables)
```
```bash
$> info ivars # Show information about instance variables about self
```
```bash
$> bt # Show backtrace (frame) information
```
```bash
$> bt 5 # Only shows first <num> frames
```
```bash
$> bt /your_method/ # Only shows frames with method name or location info that matches /regexp/
```
```bash
$> whereami # Show the current frame with source code
```
---
# Debugging with VS Code Extension
**The rdbg debugger extension for VS Code, provides a rich and convenient debugging experience.**
*Features of the VS Code Extension*

- **Interactive Debugging**: Set breakpoints, step through code, and interactively inspect variables and expressions.
- **Variable Watch**: Add variables to watch and monitor their values during debugging.
- **Call Stack Navigation**: Navigate through the call stack and view stack frames to understand program flow.
---
# cont... (features)
- **Conditional Breakpoints**: Set breakpoints that only trigger when certain conditions are met.
- **Breakpoint Management**: Easily manage and remove breakpoints through the VS Code interface.
- **Expression Evaluation**: Evaluate expressions in the Debug Console while debugging.

---
# Benefits of the VS Code Extension
- **User-Friendly Interface**: Provides familiar and intuitive debugging interface.
- **Seamless Integration**: Features are seamlessly integrated with the existing VS Code workflows, minimizing context-switching.
- **Powerful Debugging Tools**: Uses VS Code's extensive debugging tools, such as variable watching, expression evaluation, and call stack navigation.
- **Code Navigation**: Use the editor's features to navigate code, jump to definitions, and inspect code snippets while debugging.

---
# Using the console to debug
*The console can interact with your app's model to debug data issues.*

```bash
rails console # start the console
```
```bash
rdbg -n --open -c -- rails console # start the console in debug mode
```

```bash
User.last # Retrieve the last user record
```
```bash
User.where(name: 'John') # Find users with a specific attribute value
```
---
# cont...
```bash
user = User.find(1)
user.email = 'newemail@example.com'
user.save! # Update an attribute for a specific record
```
```bash
results = ActiveRecord::Base.connection.execute('SELECT * FROM users limit 10')
results.each do |row|
  puts row.inspect # Inspect first 10 rows in the users table
end
```
```bash
users = User.all
users.each do |user|
  puts user.name # run custom code tests
end
```
---
# Debugging Views in Rails
**Debugging views using the debug helper**
*Use debug to inspect Ruby objects*

```
# debugging inline
<%= debug @user %>

# multiple objects
<%= debug @user, @post, @comment %>

# with labels
<%= debug @user, label: 'User object' %>
<%= debug @post, label: 'Post object' %>
<%= debug @comment, label: 'Comment object' %>
```
---

# The benchmark gem
```ruby
require 'benchmark'
n = 5000000
Benchmark.bm(7) do |x|
  x.report("for:")   { for i in 1..n; a = "1"; end }
  x.report("times:") { n.times do   ; a = "1"; end }
  x.report("upto:")  { 1.upto(n) do ; a = "1"; end }
end
```
```
              user     system      total        real
for:      1.010000   0.000000   1.010000 (  1.015688)
times:    1.000000   0.000000   1.000000 (  1.003611)
upto:     1.030000   0.000000   1.030000 (  1.028098)
```
---
# The benchmark-memory gem
```ruby
require "benchmark/memory"

def allocate_string
  "this string was dynamically allocated"
end

def give_frozen_string
  "this string is frozen".freeze
end

Benchmark.memory do |x|
  x.report("dynamic allocation") { allocate_string }
  x.report("frozen string") { give_frozen_string }
  x.compare!
end
```
---
# Memory Benchmarking Results
```txt
Calculating -------------------------------------
  dynamic allocation    40.000  memsize (     0.000  retained)
                         1.000  objects (     0.000  retained)
                         1.000  strings (     0.000  retained)
       frozen string     0.000  memsize (     0.000  retained)
                         0.000  objects (     0.000  retained)
                         0.000  strings (     0.000  retained)

Comparison:
       frozen string:          0 allocated
  dynamic allocation:         40 allocated - Infx more
```

---

# object.send("some_function")

**Understanding Ruby's Object#send method**
*Object#send invokes a method identified by a string or a symbol. It can even call private methods.*

```ruby
class MyClass
  private

  def my_private_method
    puts 'Hello, World!'
  end
end

MyClass.new.send(:my_private_method)
```

---

# Common Pitfalls of ActiveRecord

**N+1 Query problem**
*N+1 query is a common performance pitfall. Eager loading can help avoid this problem.*

```ruby
# N+1 problem
users = User.all
users.each { |user| puts user.posts.count }

# Solution
users = User.includes(:posts)
users.each { |user| puts user.posts.count }
```
---

# Eager Loading Ruby

**Understanding eager loading of classes in Rails**
*Eager loading can improve performance by loading all associated records in a single database query.*

```ruby
# this reuturns a full ActiveRecord with all the columns populated, just to see if it exists...
if User.find_by_email_and_account_owner(params[:email], true) && region != "us"

# <>.exists?(field...) just returns true|false if it exists from the DB, way less memory
if User.exists?(email: params[:email], account_owner: true) && region != "us"

return 'has posts' if has_posts
```

---

# Class Loading in Rails
**Understanding Rails autoload and eager_load**
*Autoloading in Rails allows classes to be loaded on demand.*

```ruby
# eager_load loads all classes in the app directory.
# app/initalizers/eager_load.rb
Rails.application.eager_load!

# app/models/user.rb
user = User.new

# app/models/user.rb
class User < ApplicationRecord
  has_many :posts
end
```

---
# Class loading in rails
**How Rails decides which class to load if there are more than one with the same name**
*Rails uses the concept of "constant lookup" to resolve class names.*

```ruby
module M
  class C
    X = 'a constant'
  end
  puts C::X  # => "a constant"
end
puts C::X  # => uninitialized constant error
```
---
# Searching code to find methods/constants/classes
**Using regex can come in very handy when searching for a block of code that might have been built or called internally.**

```
# Methods that start with "find_by_<some_field>"
/^def find_by_/

# Instances of obj.send("some_method")
/obj\.send\(\".*\"\)/

# Instances of obj.send("some_method") where "some_method" is a private method
/obj\.send\(\".*\"\)/
```
---

# more regexes
```
# searching for a scope in any models
/^scope :/ # within by **app/models**

# searching for a test in a spec file
/^it 'should/ # finds all the tests in a spec file that start with "it 'should"

# searching for interpolated strings
/#{.*}/ # finds all the interpolated strings

# searching for a method that takes a block
/^def .*(&.*)/ # finds all the methods that take a block

# searching for rescue blocks with a specific error message
message = 'some error message'
/rescue #{message}/ # finds all the rescue blocks with the message
```

---
# Q/A & Live Demos
    1. Start writing ruby
    2. Many months later...
    3. Who wrote this garbage?
    4. Load up debugger
    5. ???
    6. Fix the issue
    7. Profit!
---
