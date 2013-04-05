OPBE
====

**O**game **P**robabilistic **B**attle **E**ngine  
live: http://opbe.webnet32.com

## Introduction
OPBE is the first(and only) battle engine in the world that use Probability theory:  
battles is processed very very fast and required few memory resources.  
Also memory and cpu usage are O(1), this means they are COSTANTS and independent of the ships amount.    

features:
* Multiple players and fleets (ACS)
* Fake targets
* Rapid fire
* Ordered fires
* Bounce rule
* Waste of damage
* Moon attempt
* Defenses rebuilt
* Report generated by templates
* Cross-platform (XgProyect,2moons,Xnova,Wootook etc)

---

## Accuracy
One of main concept used in this battle engine is the **expected value**: instead calculate each ship case, opbe 
creates an estimation of their behavior. 
This estimation is improved analyzing behavior for infinite simulations, so 
to test opbe accuracy you have to set speedsim(dragosim)'s battles amount to a big number, such as 3k.  

---

## Performace
Seems that no one know the O() concept, from wiki:  
*big O notation is used to classify algorithms by how they respond (e.g., in their processing time or working space requirements) to changes in input size.*   
There are different possibilities, such as O(n) (linear), O(n^2) (quadratic) etc.
OPBE is O(1) in both CPU and MEMORY usage:  
pratically, it say that the computional requirements to solve an algoritm **don't increase** with the input size.  
It's awesome! You can simulate 20 ships or 20 Tera ships and.. nothing should change. Really, there is a small difference  
cause the aritmetic operation aren't O(1) for CPU, also a *double* require more space than an *integer*.  

Let's do some test:

##### Double's limit test 
Write a lot of 9 in both defender's BC and attacker's BC. Cause double limit, your number will be trunked to *9223372036854775807*.  
Anyway the battle starts and are calculated as well(some overflow errors maybe).  

    Battle calculated in 2.54 ms.
    Memory used: 335 KB
    
##### Worst case, OPBE max requirements!
The only think that require a lot of memory in OPBE is the Report, cause it store all ships in each rounds.. this means more  
rounds = more memory (a draw).  
Also, the worst case for CPU and memory is to have all types of ships(iteration of them).
So we set 1'000'000 of ships in each type(defenses only for defender),not a big number just to avoid overflow in calculations.  
And we got:

    Battle calculated in 144.88 ms.
    Memory used: 6249 KB

Seems a lot? try a O(n) algorithm and you will crash with insane small amount of ships :)  
Anyway you can decrease a lot the memory usage setting, in *constants/battle_constants.php*,  

```php 
    define('ONLY_FIRST_AND_LAST_ROUND',true);
```

---

## Quick start
Ok, seems so cool! How i can use it?  
You can check in **implementations** directory for your game version, read the installation.txt file.  
Be sure you had read the *license* with respect for the author.

---

## Making new test cases to share
Something wrong with OPBE? The fastes way to share the simulation is to make a test case.  

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

So in a PlayerGroup there are differents Player,  
each Player have differents Fleets,  
each Fleet have differents Figthers.  

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

Fighters is the smallest object in the system: it rappresent a group of specific object type able to fight.
For some reason, opbe need to categorize it in two type extending Fighters:
* Defense
* Ship

Don't care about this fact because you should use this automatic code:

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

Note that you can assign differents techs to each Fighters, see functions inside this class. 

#### Fleet

Fleet is a group of Fighters and it is extended by a single object 
* HomeFleet : rappresent the ships and defense in the planet (of owner)

This time you have to manually choose the right class and HomeFleet should have $id = 0;

```php
    $fleet = new Fleet($idFleet); // $idFleet is a must
    $fleet = new HomeFleet(0); // 0 is a must
    $fleet->add($fighters);
```
Note that you can assign differents techs to each Fleets, see functions inside this class.   
In this case, all Fighters contained in a Fleet will take it techs.

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
The first argument of constructor is the **attacking** PlayerGroup, instead,   
the second one is the **defending** PlayerGroup.  

There are two methods needed for you:
* startBattle(boolean $debug) : start the battle, if $debug == true, informations will be writtem in output.
* getReport() : return an object usefull to retrive each type of battle informations


```php
    $engine = new Battle($attackingPlayerGroup, $defendingPlayerGroup);
    $engine->startBattle(false);
    $info = $engine->getReport();
```

#### Report

Report is a big container of informations about the battle simulated.  
In the web interface of OPBE, the instance of Report is injected in templates.   

```php
    $info = $engine->getReport();
```

Report contains all the ships status in each round, also have usefull functions to fill templates or update  
the database after the batte.

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

