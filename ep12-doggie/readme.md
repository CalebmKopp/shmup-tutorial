# ep12 - collisions

## Objectives
- understand how Box Collision is calculated
- implement it between enemy x ship

## Challenge 1
- program collision between bullets and enemies
---
- this requires, a lot of math
- LOL it actually doesn't
  - just like, take your existing enemies update for loop
  - and add in a check for every bullet on screen
  - if, there's a bullet and enemy collision
    - delete enemy
    - delete bullet
    - add obj to `deads` list
  - then just render/update all the `deads` - which, is just showing an explosion
## Challenge 2
- fix the shoot-button-hold bug
  - as of right now, when you hold the shoot button
  - it fires one bullet, and then after a bit it starts firing bullets rapidly
  - we want it such that when you hold the shoot button it fires rapidly right away
  - as well, want to be able to register every time our shoot button is pressed with exact frequency, whereas right now it's moreso whenever the game decides that `btnp(...)` is true again
---
- so, this can be fixed by editing the pico8 memory directly
  ```lua
  poke(0x5f5c, delay) -- set the initial delay before repeating. 255 means never repeat.
  poke(0x5f5d, delay) -- set the repeating delay.
  ```
- Basically there's two delays - when holding down a button and detecting something with btnp, there's a baked in 15 frame delay before a super-repeat happens
- and that super-repeat delay is 4 frames
- so, what we can do is just change the first address from `15` to `4` and that should fix the problem
## Challenge 3
- Instead of when the enemy hits your ship, you enter an invulnerability state instead of just losing all your lives instantly
---
- added 3 props to ship, `hide`,`invuln`,`invuln_t`, initialized at `false, false, 0`
- if `hide` is true, then the ship spr will become fullblack
- if `invuln` is true, it starts a 1 second timer, which is tracked in `invuln_t`
- During this 1 second, the `ship.hide` will oscillate between true and false - which "flickers" the ship sprite
- the boost sprite is always shown, letting the player still keep track of where they are
- Also experimenting with different invuln flickering
  - just a square
  - a square with an outline
  - no sprite at all (standard)
## Notes
- Collision detection is surprisingly simple, as long as your sprites are both 8px by 8px
- It becomes more difficult later, when sprites are larger, for example
- Another way to do detection, found in a youtube comment (but again, conditional that your only have 8x8 objs and sprites)
- 
  ```lua
  function is_collided(a,b)
    return abs(a.x-b.x)<8 
    and abs(a.y-b.y)<8
  end
  ```
  - This is a version of the distance formula, getting the distance between two points
  - basically, if the (x,y) of a pixel are within 8 of the (x,y) of another, then the only way that's true is if there are two 8by8 sprites, rendering on top of one another
- another youtube comment pointed out you can use the [mid(f,s,t)](https://pico-8.fandom.com/wiki/Mid) function to keep the ship's coordinates within the play area
  - Mid on it's own takes in three numbers, and designed to return the middle of the three
  - example: `mid(23,44,55)` would return `44`
  - `mid(100,2,1000)` would return `100` - as middle is not determined by argument order but greater than or equal to comaprisons
  - it's most useful for *Clamping a value to a specific range*
  - like `ship.x=mid(0,ship.x,120)`
  - this makes it so that the `ship.x` property will always stay between 0 and 120
  - since, if it goes below `0`, `0` becomes the middle value and if it goes above `120`, then `120` becomes the middle value