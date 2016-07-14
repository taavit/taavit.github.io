---
layout: post
title:  "Processing measurements and Value Objects"
date:   2016-07-12 19:33:11 +0200
categories: design
comments: true
---

When I've started processing some measured data, some calculation, numbers I had to do this with some precision. There was speed, distance, temperature etc. All of those can be represented by double type, so using this type is natural. At least it was. Well, I still wanted to keep precise values, but some drawback appears.

Let say there is some value represent temperature, this value is 100. OK, but 100 what? Well, when talking about scientific measurements we can assume those as kelvins. 100 means 100K, sounds OK, let's put some comment to the code explaining unit of value. Sounds pretty easy.

Second step is to present this value to the user. Users prefers values in Celsius, or Fahrenheit. OK. easy before presenting we are just putting some switch/if to distinguish unit, then subtract 273.15 or subtract and multiply and we are done. User now see temperature in his favourite unit.

And now our application depend on those temperatures, we are gathering, we are calculating, and we are presenting values in two different units, kelvins for computation because that’s our contract, and Celsius/Fahrenheit for user interaction. It might be consistent, but at some point there will be more, and more expression -273.15, copying and pasting this all the time. Copying and pasting, typing ... If there is one, or two, it's not a big deal.

Problems comes at handling tens of such temperatures, easy to make typo in those expressions while typing (hey, I'm not going to copy that, it's just ~~-271.35~~-273.15!)

Yes, our test will catch it, earlier or later, but can we just eliminate such mistakes, and clean a little our code?

Temperature value is fixed, regardless what unit is used, it's represents still the same temperature, it's not getting colder or warmer if we convert this to kelvin, Fahrenheit or Celsius. 20C converted to kelvin is 293.15K, but that is exactly the same.

Solution would be Value Object. We can put handling all units inside such object, internally we could use kelvin, but such object would contain method to represent this temperature as kelvin, or as Fahrenheit or as Celsius. Of course it will be nice to allow to create such object from different unit, But through whole process this is just an immutable single object, uses different representation for different cases. Single value, with different representation.

Let's take a look how this object might looks like.

```php
<?php

final class Temperature
{
    // private constructor to keep factory method consistent, fromKelvin, fromFahrenheit, fromCelsius
    private function __construct(float $kelvin)
    {
        $this->value = $kelvin;
    }

    public static function fromKelvin(float $kelvin): Temperature
    {
        return new self($kelvins);
    }

    public static function fromFahrenheit(float $fahrenheit): Temperature
    {
        return new self(($fahrenheit - 32) / 1.8 + 273.15);
    }
}
```

Distance, speed are similar cases. Distance and speed aren't change depend how we want to represent them, those are the same. 

```php
<?php

final class Distance
{
    // private constructor to keep factory method consistent, fromMeters, fromFeets
    private function __construct(float $meters)
    {
        $this->value = $meters;
    }

    public static function fromMeters(float $meters): Distance
    {
        return new self($meters)
    }

    public static function fromFeet(float $feet): Distance
    {
        return new self($feet / 3.2808);
    }
}
```

In most cases, making such object might be an overhead, but in more complex systems, where there is a lot interaction between users and system, this approach might help working with code, and we can forget in which unit we have this float.
