#Map, Filter and Reduce IRL

I've seen several blog posts and articles trying to explain what map, filter and reduce are and
praising the advantages of functional programming in JavaScript, however, I still have students and
friends that had a hard time understanding or seeing the application; I found that using real examples
from a codebase or an application is really easy to get it.

For this example application, we're going to have a list of products that a customer wants to buy,
we'll calculate the tax if it applies, shipping, the total and separate them on categories; will include the code for both
the functional version and classic version so that the advantage is clear.

##Initial data

This is the product information that will be used for all the examples. This functions work
with arrays.

Let's pretend we have a computer parts store and this customer wants to buy a console killer.

```
var products_in_cart = [
  {name: 'AMD Athlon X4 860K 3.7GHz Quad-Core Processor', unit_price: 89.98, type: 'CPU', brand: 'AMD', weight: 2, amount: 1},
  {name: 'Cooler Master Hyper T2 54.8 CFM Sleeve Bearing CPU Cooler', unit_price: 15.99, type: 'COOLER', brand: 'Cooler Master', weight: 2, amount: 1},
  {name: 'Gigabyte GA-F2A88XM-D3H Micro ATX FM2+ Motherboard', unit_price: 84.88, type: 'MOTHERBOARD', brand: 'Gygabyte', weight: 2, amount: 1},
  {name: 'G.Skill Ripjaws X Series 8GB (1 x 8GB) DDR3-1600 Memory', unit_price: 29.99, type: 'RAM', brand: 'G.Skill', weight: 1, amount: 1},
  {name: 'Seagate Barracuda 1TB 3.5" 7200RPM Internal Hard Drive', unit_price: 46.89, type: 'HDD', brand: 'Seagate', weight: 2, amount: 1},
  {name: 'Radeon™ RX 480 Graphics Card', unit_price: 199, type: 'GPU', brand: 'AMD', weight: 2, amount: 1},
  {name: 'Thermaltake Versa H21 ATX Mid Tower Case', unit_price: 35.12, brand: 'Thermaltake', weight: 3, amount: 1},
  {name: 'EVGA 500W 80+ Certified ATX Power Supply', unit_price: 39.90, brand: 'EVGA', weight: 2, amount: 1}
];
```
##Calculating the price before tax using reduce

For the first part, let's see how much money does the customer have to pay before taxes and shipping costs.
Using a classical for loop the code would look like this:

```
var price_before_tax = 0;
for (var i = 0; i < products_in_cart.length; i++) {
    price_before_tax += (products_in_cart[i].unit_price * products_in_cart[i].amount);
}
```

Not so bad, but it can look better with reduce.

```
var price_before_tax = products_in_cart.reduce(function (p, c) {return p + (c.unit_price * c.amount); }, 0);
```

The reduce method applies a function to each value of an array to reduce it to a single value. The
first argument is the callback that will be applied to calculate the new values and the second
argument is the initial value that will be calculated.

##Calculating the tax for each product with map

Most states don't require you to charge sale taxes from online sales, but some states do, let's pretend
that our customer is in California, and that sale tax in CA is 9%. Using a for loop, it would look like this:

```
var cart_with_taxes = [];
// pretend user information
var user = {location: 'CA'};
for (var i = 0; i < products_in_cart.length; i++) {
    cart_with_taxes[i] = products_in_cart[i];
    cart_with_taxes[i].price_with_tax = (products_in_cart[i].unit_price * products_in_cart[i].amount);
    if (user.location === 'CA') {
        cart_with_taxes[i].price_with_tax = (products_in_cart[i].unit_price * products_in_cart[i].amount) * 1.09;
    }
}
```

```
var user = {location: 'CA'};
var cart_with_taxes = products_in_cart.map(function (a) {
    a.price_with_tax = (c.unit_price * c.amount);
    if (user.location === 'CA') {
        a.price_with_tax = (c.unit_price * c.amount) * 1.09;
    }
    return a;
});
```

The map method creates a new array with the results of calling a provided function on every element
in the array.

##Applying discounts to some products with filters

Our customer received a coupon from AMD during E3, this coupon will give her a 20% discount on all
AMD products. Using a for loop, it would look like this:

```
var cart_with_discounts = [];
var discounted_products = [];
for (var i = 0; i < products_in_cart.length; i++) {
    var discounted_price;
    cart_with_discounts[i] = products_in_cart[i];
    discounted_price = products_in_cart[i].price * products_in_cart[i].amount;
    if (products_in_cart[i].brand === 'AMD') {
        discounted_price = discounted_price * 0.8;
        discounted_price.push(products_in_cart[i]);
    }
    cart_with_discounts[i].discounted_price = discounted_price;
}
```
The filter way is simpler and easier to read.
```
var discounted_products = products_in_cart.filter(function (a) {
    return a.brand === 'AMD'
});
```
The filter() method creates a new array with all elements that pass the test implemented by the
provided function.

##All together

Let's put all the things together and take a look at how they look.

This methods can be chained, have in mind the return values of each function to know what
can be used in the following. For example, reduce returns a single value, so other functions
can't be chained to it. Filter will return an array with only the matching elements, so the following
functions won't have access to all the elements to perform calculations on.

```
var products_in_cart = products_in_cart.map(function (a) {
    // calculate total price
    return a.total_price = a.unit_price * a.amount;
}).map(function (a) {
    // add discounts to the calculation
    if (a.brand === 'AMD') {
        a.total_price = a.total_price * 0.8;
    }
}).map(function (a) {
    // add the tax to the price
    if (user.location==='CA') {
        a.price_with_tax = a.total_price * 1.09;
        return a;
    }
    return a;
});
```
Now the `products_in_cart` variable is transformed to have all the calculations and information
necessary for the application to show the customer and manage all of it.

