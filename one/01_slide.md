!SLIDE 
# 4 Steps to Faster Rails Test
### Tom Clements
### _Senior Developer, On The Beach_
<br />
<br />
<br />
### tom-clements.com | github.com/seenmyfate
### @Seenmyfate
!SLIDE bullets incremental
#A question

* How long does it take to run your unit tests?

!SLIDE commandline incremental
#Let's look at some numbers

    $ time rake spec
     31.486 seconds
    
    $ time bundle exec rspec spec
     15.741 seconds
    
    $ time rspec spec
     14.932 seconds

!SLIDE
# And what am I testing? #

!SLIDE
# Nothing!

      @@@ ruby
      # spec/nowt_spec.rb
       require 'spec_helper'

       describe "Nothing" do
          # nada
       end

       0 examples, 0 failures
       15.741 seconds total

!SLIDE incremental
# So how can we make this better?

!SLIDE bullets incremental
## Here's some common suggestions

* Don't Test
* Get an SSD
* Use Spork

!SLIDE bullets incremental
#these solutions are just masking the problem

* And the first part of that problem is spec_helper

!SLIDE
_spec helper is the best way to write really slow painful specs_

Gary Bernhardt

!SLIDE
## With spec_helper - 15.741 seconds

## Without:

!SLIDE
# 0.186 seconds
    @@@ ruby
    # spec/some_spec.rb
    # require 'spec_helper'

    describe "Nothing" do
       # nada
    end

    0 examples, 0 failures
    0.186 seconds total

!SLIDE
# So here's the question

!SLIDE bullets incremental
## For our unit tests - 
## Why are we loading the entire rails environment?

* Because our business logic is in our active record models
* And we need rails to run tests against classes that inherit from ActiveRecord::Base
* Because we're breaking SRP

!SLIDE bullets incremental
# Single Responsibility Principle

* A class should only have one reason to change

!SLIDE bullets incremental
# 4 Steps

* Extract business logic into modules
* Extract domain objects into classes
* Mixin and delegate
* Test in isolation

!SLIDE
# An example
    @@@ ruby
    class Basket < ActiveRecord::Base
      has_many :basket_items

      def total_discount
        basket_items.collect(&:discount).sum
      end
    end
    
!SLIDE smaller
# An example
    @@@ ruby
    require 'spec_helper'
    describe Basket do
      context "total_discount" do
        let(:basket) { Basket.create! }
        let(:basket_items) { [BasketItem.create!(:discount => 10),
                              BasketItem.create!(:discount => 20) ]}

        it "should return the total discount" do
          basket = Basket.create!
          basket.basket_items = basket_items
          basket.total_discount.should == 30
        end
      end
    end

!SLIDE
# 7.092 seconds to run

    time rspec spec
    .

    Finished in 0.24435 seconds
    1 example, 0 failures
    rspec spec 7.092 total    
        
!SLIDE
# Extract behaviour into modules
    @@@ ruby
    module DiscountCalculator
      def total_discount
        basket_items.collect(&:discount).
          inject(:+)
      end
    end

!SLIDE
# Mixin
    @@@ ruby
    class Basket < ActiveRecord::Base
      has_many :basket_items
      include DiscountCalculator
    end
    
!SLIDE smaller
# And the test
    @@@ ruby
    require 'discount_calculator'

    class FakeBasket
      include DiscountCalculator
    end

    describe DiscountCalculator do

      context "#total_discount" do
        it "should return the total discount" do
          basket = FakeBasket.new
          basket_items = [stub(:discount => 10),
                          stub(:discount => 20)]
          basket.stub(:basket_items) { basket_items }
          basket.total_discount.should eq 30
        end
      end
    end

!SLIDE
# 0.350 seconds to run

    time rspec spec
    .

    Finished in 0.00121 seconds
    1 example, 0 failures
    rspec spec 0.350 total

!SLIDE
# Extract domain objects into classes
    @@@ ruby
    class DiscountCalculator
      def total_discount(items)
        items.collect(&:discount).inject(:+)
      end
    end
    
!SLIDE
# And delegate
    
    @@@ ruby
    class Basket < ActiveRecord::Base
      has_many :basket_items
      
      def total_discount
        DiscountCalculator.new.
          total_discount(basket_items)
      end
    end

    

!SLIDE smaller
# And the test
    @@@ ruby
    require 'discount_calculator'
    describe DiscountCalculator do
      context "#total_discount" do
        let(:items) { [stub(:discount => 10),
                       stub(:discount => 20)] }
        it "should return the total discount" do
          calculator = DiscountCalculator.new
          calculator.total_discount(items).should eq 30
        end
      end
    end

!SLIDE
# 0.342 seconds to run

    time rspec spec
    .

    Finished in 0.00101 seconds
    1 example, 0 failures
    rspec spec 0.342 total

!SLIDE bullets incremental
## The benefits

* Higher Cohesion
* Lightning fast tests
* Happier developing
* Try it out
* github.com/seenmyfate

!SLIDE
# Further viewing/reading

@coreyhaines - confreaks.net/videos/641-gogaruco2011-fast-rails-tests

@garybernhardt - destroyallsoftware.com

@martinfowler - objectmentor.com/resources/articles/srp.pdf

!SLIDE 
# 4 Steps to Faster Rails Test
### Tom Clements
### _Senior Developer, On The Beach_
<br />
<br />
<br />
### tom-clements.com | github.com/seenmyfate
### @Seenmyfate

