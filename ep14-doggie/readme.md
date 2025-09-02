# ep14 - explosions

## Objectives
- learn about making explosions
- learn about using the `pal()` function

## Challenge 1
- use `pal()` function to create different color enemies on screen, without drawing more sprites for those enemies
## Challenge 2
- when player is hit, make the ship flash (using pallette swapping) instead of oscillating on/off
## Challenge 3
- when a bullet hits an enemy, make a small contact explosion - nothing fancy 
  - I think the way i wanna do this, is with `circfill()`
  - basically, when a bullet is about to be deleted due to contact with an enemy, draw circles that are bullet colored, then draw a black circle in the middle that expands out to subsume that bullet impact circle
  - since, i am silly and i have these cute animated oscillating bullets, could try to do double contact but let's just start with one for now
  - so, i want to freeboot this kind of code
  ```lua
  --_draw()
	if muzzle>0 then
		circfill(ship.x+3,ship.y-2,muzzle,7)
		circfill(ship.x+4,ship.y-2,muzzle,7)
	end
  ```
  - mini explosion drawing
  ```lua
  --_update()
  ...
  if col(bul_ref,en_ref) then
    del(buls, bul_ref)
    add(minis,{x=bul_ref.x,y=bul_ref.y,life=4})
  ...
  --_draw()
	for m in all(minis) do
		if m.life%2==0 then
			pal(11,3)
		end
		circfill(m.x+3, m.y, 2, 11)
		circfill(m.x+4, m.y, 2, 11)
		rectfill(m.x+3, m.y-1, m.x+4, m.y+1, 7) 
		pal()
		m.life-=1
		if (m.life <= 0) then
			del(minis,m)
		end
	end
  ```
  - when bullet hit enemy, we add a mini explosion with 4 frames of life
  - when we render that list, we pallette swap rapidly and then it disappears
  - good enough for now lol
## Notes
- "An explosion must always been bigger than the thing it explodes" important axiom
- maybe next episode, is make explosions procedurally - since, when we create explosions manually in the spritesheet with a sprite index array and manual sprite choices in that array, it takes up a lot of space, tokens, and time/effort
- drawing explosions on dead enemies
    ```lua
        --_draw()
        ...
        --draw booms
        local boom_ages={64,64,64,66,68,68,70,72,72}
        for boom_ref in all(booms) do
            --local variable for some
            -- readability
            local b={
                x=boom_ref.x,
                y=boom_ref.y,
                w=2,
                h=2,
                spr=boom_ages[flr(boom_ref.age)]
            }	
            
            drw_obj(b)
            
            boom_ref.age+=1 --can be decimal, since it's flr()'d above
            if (boom_ref.age>#boom_ages) then
                del(booms, boom_ref)
            end
    ```
    - we have code here, that for all booms that exist (which, is just an x/y coord, and an age)
      - we make a local obj that is compatible with our `drw_obj(...)` function
      - and then draw that obj (utilizing a custom width and height as well)
      - as well, we're advancing its age by one, and if its age is ever higher than the number of sprites in its ages list, then we delete the boom
    - this way, we can control how long each sprite of the boom is on screen
      - this can also be controlled by the age progression
    - Specifically, we have a calculation to get the appropriate sprite for a boom of a specific age
      - `spr=boom_ages[flr(boom_ref.age)]`
      - we take the age, and `flr()` it, so that any decimals are rounded down to the nearest whole integer
      - then, we take the index of `boom_ages[int]` at that integer and make that our sprite index
      - which is then drawn
    - this takes a lot of manual work and coding and thinking, and can be done procedurally (maybe using `circfill()`)