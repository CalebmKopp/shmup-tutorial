## Challenge 1
- Expanding the bullet object by adding properties - try having bullets with different speeds
- or try to have bullets that change sprites as they fly
- OR WHAT IF THE BULLETS ARE ANIMATED (bullets having a frame property - which i am 100000% going to do and already planned to do)
  - this is done, our bullets animate independently while flying upward
## Challenge 2
- can you do something like a Spreadshot? when you shoot the bullet, can you also shoot two more that go diagonally?
  - this is complete, but only inthe `spreadshot.p8` file since it does re-arrange a lot of code
## Notes
- if something breaks, that's okay! you're testing limits
- When it comes to deleting bullets, we can look at the following code
    ```lua
        --move the bullets
        for i=1, #buls do
            local bul_ref=buls[i]
            bul_ref.y-=bullspd
            if should_ani_buls then
                ani_bul(bul_ref)
            end
            if bul_ref.y<-10 then
                del(buls,bul_ref)
            end
        end
    ```
  - In its current form, this for loop will start at `index1` and loop through each bullet in the list - `index2`, `index3` etc
  - when a bullet is too far off screen so as not to be visible in the playspace, we should delete that bullet reference in the array
  - freeing up memory, and making it so that we don't overload the engine with too many objects way down the line with a long play session
  - However, this code has a problem - lets track the scenario of 3 bullets fired, 20 pixels away from the top of the screen, going -7 pixels per frame, and when the bullet goes fully beyond the screen we want to delete it from the array
    - frame 1 - bullet1 has y being 20, bullet2 and bullet3 do not exist
    - frame 2 - bullet1 has y being 13, bullet2 has y being 20, and bullet3 does not exist
    - frame 3 - bullet1.y== 6, bullet2.y== 13, and bullet3.y== 20
    - frame 4 - bullet1.y== -1, bullet2.y== 6, bullet3.y== 13
    - frame 5 - bullet1.y== -8
      - therefore, in our for loop we delete bullet1
    - frame 5 (cont) - bullet2.y== 6, bullet3.y???
    - **ERROR**
  - Now, we have an error because bullet3 no longer exists, after we deleted bullet1
  - in doing this array deletion operation, we're reducing the size of the #buls array
  - HOWEVER, we're doing this in the middle of the for loop - when this for loop started it just knew it needed to run three times, and that it expected a `buls[3]` to exist
  - but since `bullet3` just became `bullet2` via our deletion process, the real `bullet2` just became `bullet1` - as everything shifts when an array obj is deleted or added
  - and, since we're not looking at `buls[1]` anymore - it actually skipped of frame of animation! The second bullet we fired, would stay in place and not get an updated y value, due to its new home in the array at `index1`. 
    - This what's called a `Logical Error` - in that we wrote the code and it executed without crashing, but we got an undesired or unexpected result (the second bullet we fired stayed in place for a frame) 
  - As well as this, the more error that we are attempting to access `index3` of the `buls{}` list when it does not have an obj in `index3` causes the program to crash, when we try to do the operation `buls[3].y=6` since, `nil does not have property of y`
    - This is what's called a `Runtime Error` in that we wrote code, and the program started without crashing, but while running something unexpected or impossible happened which caused the execution of our code to cease (trying to essentially call `nil.y` because `buls[3]==nil`)
- So how do we fix this? a few approaches
  - you could precheck in the forloop, that if a `bul_ref` is nil, then skip trying to touch it at all
    ```lua
        --move the bullets
        for i=1, #buls do
            local bul_ref=buls[i]
            if bul_ref==nil then
                goto continue
            end
            bul_ref.y-=bullspd
            if should_ani_buls then
                ani_bul(bul_ref)
            end
            if bul_ref.y<-10 then
                del(buls,bul_ref)
            end
            ::continue::
        end
    ```
    - though, using `goto` in any fashion is quite obtuse, even if this isa fairly supported pattern
    - as well, when we end up still getting the same `Logical Error` in that the second bullet we fired, even just for one frame, does not get its y value updated, therefor it does not move and actually we end up with two bullets being rendered on the same coordinate
    - in the framecount example, the *italicized* bullet is the third one we fired, and since `bullet1` gets deleted, that third bullet we fired gets downshifted to `bullet2` and the second bullet we fired gets downshifted to `bullet1`
      - frame 4 - bullet1.y== -1, bullet2.y== 6, *bullet3*.y== 13
      - frame 5 - bullet1.y== -8 - delete, *bullet2*.y== 6, bullet3(`nil` - so do nothing)
      - frame 6 - bullet1.y== -1, *bullet2*.y== -1
      - frame etc...
  - what's suggested in the video is simply doing some critical thinking about what order bullets are processed in; make the for loop iterate backwards
    ```lua
        --move the bullets
        for #buls,1,-1 do
            local bul_ref=buls[i]
            bul_ref.y-=bullspd
            if should_ani_buls then
                ani_bul(bul_ref)
            end
            if bul_ref.y<-10 then
                del(buls,bul_ref)
            end
        end
    ```
  - Taking a look at the frame simulation counter, lets just replay the last few frames (remembering, that we're processing bullets in reverse order now)
    - frame 4 - bullet3.y== 13, bullet2.y== 6, bullet1.y== -1
    - frame 5 - *bullet3*.y== 6, bullet2.y== -1, bullet1.y== -8 
      - therefor, delete bullet1
    - frame 6 (for #buls,1,-1) - *bullet2*.y== -1, bullet1== -8
      - therefor, delete bullet1
    - well, our #buls is 2 now, not three
    - so, it can process whats in the list just fine without ever worrying about accessing something out the bounds that would be nil
    - as well, the third bullet we fired has been downshifted to `index2`, but it because of the update order we adjust this bullet, and don't skip the bullet we fired before it in sequence
- This also can be logically expanded, to delete bullets that aren't in a linear order
  - for example, when *shooting a spreadshot*, bullets1,2,3 all come out at the same time, but likely leave the screen at different times
  - or for example, *hitting an enemy!* this would need to un-render a bullet, and likely the bullets fired before and after it still need to be displayed
  - if bullet3, and bullet2, leave the screen before bullet1 - these deletes of an index higher than 1 means bullet1 doesnt have to shift and is processed in the righter order
  - if bullet 3, and bullet1, leave the screen before *bullet2* - you delete bullet3, bullet2 has not shifted, you delete bullet1, and then you start again from the top, which would be the *second bullet fired*, but at `index1` now