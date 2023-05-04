---
categories: Ruby
name: Prefer shortened namespaces
---

Ruby provides a way to reference other classes and modules using relative references from within the current namespace or any parent namespace, which allows you to omit the leading namespace qualifier.
Shortened names tend to be more readable specially when we have long namespaces.

## Bad

````ruby
module SomeArea
  module SomeApp
    module SomeComponent
      class MyClass
        def my_method
          SomeArea::SomeApp::MyOtherClass.new
        end
      end
    end
  end
end
````

## Good

````ruby
module SomeArea
  module SomeApp
    module SomeComponent
      class MyClass
        def my_method
          MyOtherClass.new
        end
      end
    end
  end
end
````
