# Modifying the boot process

Running `zeus init` creates a default [`zeus.json`](/burke/zeus/tree/master/examples/zeus.json) that defines the boot order for your application. Chances are not all of it is relevant -- for example, it includes commands for both test/unit and rspec. These can be removed from the JSON file and they will not be booted.

Each node under "plan" in the JSON file has an action associated with it. You can see the "command" value at the top of the hash requires `"zeus/rails"`. If you look at [that file](/burke/zeus/tree/master/rubygem/lib/zeus/rails.rb), you will see a module with a method name corresponding to each node -- something along the lines of:

```ruby
class Zeus::Rails < Zeus::Plan
  def boot
    # ...
  end
  def default_bundle
    # ...
  end
  # ...
end
Zeus.plan = Zeus::Rails.new
```

Note that an instance of this class is assigned to `Zeus.plan`. Zeus calls methods on `Zeus.plan` to boot the application. If you follow any path to a leaf node in the tree -- for example, boot, default_bundle, development_environment, prerake -- those methods are essentially called in sequence to construct an environment for a command (rake, in this case) to run in. Zeus forks the ruby process between each step, and can restart from any of these forks.

If you wanted to create your own plan, you could subclass `Zeus::Rails`:

```ruby
# /path/to/my_app/lib/my_zeus_plan.rb
require 'zeus/rails'
class MyPlan < Zeus::Rails
  def boot
    something_else
  end
end
Zeus.plan = MyPlan.new
```

```javascript
// /path/to/my_app/zeus.json
{
  "command": "ruby -r./lib/my_zeus_plan.rb -eZeus.go",
  "plan": {
    // ...
  }
}
```

Alternatively, you could subclass `Zeus::Plan` to create a completely fresh plan with no predefined behaviour.
