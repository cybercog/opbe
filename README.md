OPBE
====

**O**game **P**robabilistic **B**attle **E**ngine  
live: http://opbe.webnet32.com (server offline)

1. [Introduction](#introduction)
2. [Quick start](#quick-start-installation)
3. [Accuracy](#accuracy)
4. [Performance](#performance)
5. [TestCases](#making-new-test-cases-to-share)
6. [Developers guide](#implementation-developing-guide)
7. [License](#license)
8. [Questions](#questions)
9. [Who is using OPBE](#who-is-using-opbe)

## Introduction
OPBE is the first (and only) battle engine in the world that uses Probability theory:  
battles are processed very very fast and required little memory resources.  
Also, memory and CPU usage are O(1), this means they are CONSTANT and independent of the ships amount.    

*Features:*
* Multiple players and fleets (ACS)
* Fake targets (Fodder)
* Rapid fire
* Ordered fires
* Bounce rule
* Waste of damage
* Moon attempt
* Defenses rebuilt
* Report generated by templates
* Cross-platform (XgProyect,2moons,Xnova,Wootook etc)

---

## Quick start (installation)
This seems so cool! How can I use it?  
You can check in [implementations directory](https://github.com/jstar88/opbe/tree/master/implementations) for your game version, read the installation.txt file.  
Be sure you have read the *license* with respect for the author.

---

## Accuracy
One of main concepts used in this battle engine is the **expected value**: instead calculate each ship case, OPBE 
creates an estimation of their behavior. 
This estimation is improved by analyzing behavior for infinite simulations, so, 
to test OPBE's accuracy you have to set speedsim (or dragosim)'s battles amount to a big number, such as 3k.  

---

## Performance
It seems that noone knows the O() notation, so this is the definition from the Wiki:  
*big O notation is used to classify algorithms by how they respond (e.g., in their processing time or working space requirements) to changes in input size.*   
There are different possibilities, such as O(n) (linear), O(n^2) (quadratic) etc.
OPBE is O(1) in both CPU and MEMORY usage:  
In practice, that means that the computional requirements to solve an algoritm **don't increase** with the input size.  
It's awesome! You can simulate 20 ships or 20 Tera ships and nothing should change. In reality, there is a small difference  
because the arithmetic operations aren't O(1) for CPU, and also a *double* requires more space than an *integer*.  

Let's do some test:

##### Double's limit test 
Write a lot of nines in both defender's BC and attacker's BC. Because of the *double* limit, your number will be trunked to *9223372036854775807*.  
Anyway the battle starts and are calculated as well (some overflow errors maybe).  

    Battle calculated in 2.54 ms.
    Memory used: 335 KB

##### Worst case, OPBE max requirements!
The only thing that requires a lot of memory in OPBE is the Report, because it stores all ships in every round... this means that more  
rounds = more memory.  
Also, the worst case for CPU and memory is to have all types of ships(iteration of them).
So we set 1'000'000 of ships in each type(defenses only for defender),not a big number just to avoid overflow in calculations.  
And we got:

    Battle calculated in 144.88 ms.
    Memory used: 6249 KB

Seems a lot? Try a O(n) algorithm and you will crash with insane small amount of ships :)  
Anyway you can decrease a lot the memory usage setting, in *constants/battle_constants.php*,  

```php 
    define('ONLY_FIRST_AND_LAST_ROUND',true);
```

---

## Making new test cases to share
Something wrong with OPBE? The fastest way to share the simulation is to make a test case.  

1. Set your prefered class name,but it must extends **RunnableTest**.
2. Override two functions:
  * *getAttachers()* : return a PlayerGroup that rappresent the attackers
  * *getDefenders()* : return a PlayerGroup that rappresent the defenders

3. Instantiate your class inside the file.
4. Put the file in opbe/test/runnable/

An example:

```php 
<?php
require ("../RunnableTest.php");
class MyTest extends RunnableTest
{
    public function getAttachers()
    {
        $fleet = new Fleet(1,array(
            $this->getFighters(206, 50),
            $this->getFighters(207, 50),
            $this->getFighters(204, 150)));
        $player = new Player(1, array($fleet));
        return new PlayerGroup(array($player));
    }
    public function getDefenders()
    {
        $fleet = new Fleet(2,array(
            $this->getFighters(210, 150),
            $this->getFighters(215, 50),
            $this->getFighters(207, 20)));
        $player = new Player(2, array($fleet));
        return new PlayerGroup(array($player));
    }
}
new MyTest();
?>
```

---
## Implementation developing guide 
The system organization is like :
* PlayerGroup
   * Player1
   * Player2
      * Fleet1
      * Fleet2
         * Fighters1
         * Fighters2

So in a PlayerGroup there are different *Player*s,  
each Player have different *Fleet*s,  
each Fleet have different *Figthers*.  

An easy way to display them:
```php   
    $fleet = new Fleet($idFleet);
    $fleet->add($this->getFighters($id, $count));
    
    $player = new Player($idPlayer);
    $player->addFleet($fleet);
    
    $playerGroup = new PlayerGroup();
    $playerGroup->addPlayer($player);
```

#### Fighters

Fighters is the smallest unit in the system: it reppresents a group of specific object type able to fight.
For some reasons, OPBE needs to categorize it in either one of these two types extending Fighters:
* Defense
* Ship

You shouldn't ened to care about this fact because this automatic code will do it for you:

```php
    $fighters =  $this->getFighters($idFighters, $count);
```    

```php
   
function getFighters($id, $count)
{
    global $CombatCaps, $pricelist;
    $rf = $CombatCaps[$id]['sd'];
    $shield = $CombatCaps[$id]['shield'];
    $cost = array($pricelist[$id]['metal'], $pricelist[$id]['crystal']);
    $power = $CombatCaps[$id]['attack'];
    if ($id >= SHIP_MIN_ID && $id <= SHIP_MAX_ID)
    {
        return new Ship($id, $count, $rf, $shield, $cost, $power);
    }
    return new Defense($id, $count, $rf, $shield, $cost, $power);
}
   
```

- Note that you can assign different technology levels to each Fighter, see functions inside this class. 

#### Fleet

Fleet is a group of Fighters and it is extended by a single object 
* HomeFleet : represent the ships and defense in the target's planet

This time you have to manually choose the right class and HomeFleet should have $id = 0;

```php
    $fleet = new Fleet($idFleet); // $idFleet is a must
    $fleet = new HomeFleet(0); // 0 is a must
    $fleet->add($fighters);
```
Note that you can assign differents techs to each *Fleet*, see functions inside this class.   
In this case, all Fighters contained in a *Fleet* will take it techs.

#### Player

Player is a group of Fleets, don't care about the question attacker or defender.

```php
    $player = new Player($idPlayer); // $idPlayer is a must
    $player->addFleet($fleet);
```

#### PlayerGroup

PlayerGroup is a group of Player, don't care about the question attacker or defender.

```php
    $playerGroup = new PlayerGroup();
    $playerGroup->addPlayer($player);
```

#### Battle

Battle is the main class of OPBE.  
The first argument of constructor is the **attacking** PlayerGroup, the second one is the **defending** PlayerGroup.  

There are two methods you need to know about:
* startBattle(boolean $debug) : start the battle, if $debug == true, informations will be written in output.
* getReport() : return an object useful to retrieve all kinds of battle information.


```php
    $engine = new Battle($attackingPlayerGroup, $defendingPlayerGroup);
    $engine->startBattle(false);
    $info = $engine->getReport();
```

#### Report

Report is a big container of data about the simulated battle.  
In the web interface of OPBE, the instance of Report is injected in templates.   

```php
    $info = $engine->getReport();
```

Report contains all the ships status in every round, and also has useful functions to fill templates or update  
the database after the battle.

---

## License

![license](http://www.gnu.org/graphics/agplv3-155x51.png)  
    
    Copyright (C) 2013  Jstar

    OPBE is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as
    published by the Free Software Foundation, either version 3 of the
    License, or (at your option) any later version.

    OPBE is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU Affero General Public License for more details.

    You should have received a copy of the GNU Affero General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.
    
    [why affero GPL](http://www.gnu.org/licenses/why-affero-gpl.html)

---

## Questions

> I would like to include your battle engine but the code of my game won't be published

You can keep secret your code because OPBE can be seen as a external module with own license, it's only required to share changes on OPBE.

> I would like to have some profits with my game 

GNU affero v3 accepts profits as well

> I would like use OPBE with numbers greater than double

You should replace any PHP native mathematical with [BC math](http://php.net/manual/en/ref.bc.php) functions

---
    

## Who is using OPBE?

I'm happy to deliver this software giving others the possibility to have a good battle engine.  
On the other hand, it's a pleasure to see people using my OPBE.  
Send me an email to put your game logo here!  

![xgproyect](http://www.xgproyect.net/images/misc/xg-logo.png)

