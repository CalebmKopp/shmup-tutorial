# ep11

## Objectives
- work out Enemies
- talk about Collision detection

## Challenge 1
- Spawn 5 enemies, 10 enemies - just try to spawn multiple at once
- Try to make it so they don't spawn, all at once on screen but come down in like a pattern (a line, or from a diagonal or making a loop pattern then flying off screen)
## Challenge 2
- What does it look like to spawn a second type of enemy?
- encoding different `kind` data into enemies, which then changes how they look, maybe move, maybe animate
## Challenge 3
- Add property to enemy, that you can tell from the naked eye, that makes it different
- maybe `xspd,yspd` that is different when enemies are added to array, making it fly in different ways than others
- maybe a property that governs whether it dies or not `isdead`, and when that's true a different animation plays and then its removed from the enemy array
## Notes
- Probably do these challenges, in exactly reverse order
- next episode we're going to work on collision detection - and it's a lot easier to work with this concept when you have obj's that are similar to each other
- in this case, we made `buls, enemies, ship` essentially implement the same `Interface` or `Extend` the same class, if we're going to use C terms which Lua compiles down to
  - they all have `spr, x, y` property
  - this makes them all drawable without having to repeat and copy/paste code, just calling a function to render an obj that it expects to have those three props
  - we can use this for our UI elements as well later
  - and even extend it to be something like `drw_obj_lst(...)` that accepts a list of objects that fill this shape, and iterates through them to be rendered