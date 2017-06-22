# DIY Flappy Bird - Game Tutorial
This material is based on the Youtube tutorial:
[How to make Games: Flappy Bird](https://www.youtube.com/playlist?list=PLZm85UZQLd2TPXpUJfDEdWTSgszionbJy) by @BrentAureli

## libGDX and Android Studio Setup
1. Open the libGDX wizard tool (gdx-setup.jar) and generate a new project with the following attributes:
<br>*Name:* `FlappyDemo`
<br>*Package:* `com.missionbit.game`
<br>*Game class:* `FlappyDemo`
<br>*Destination:* `/Users/missionbit/Desktop/Projects/FlappyDemo`
<br>*Android SDK:* `/Users/missionbit/Library/Android/sdk`
<br>*Sub Projects:* Select `Desktop` and `Android`
<br>*Extensions:* Select `Box2d`
<br>*Advanced:* Select `IDEA` and click `Save`
> *Hint:* The Java convention is to use all lowercase for package names, but class names are always capitalized

2. Click `Generate` and wait until the console says “BUILD SUCCESSFUL”
3. Close the window and open Android Studio.
4. `Import project` >>  `/Users/missionbit/Desktop/Projects/FlappyDemo` >> `OK`
5. Navigate to `Edit Configuration` >> Click the `+` button >> Select `Application` >> Fill out the following fields:
<br>*Name:* `Desktop`
<br>*Main class:* `com.missionbit.game.desktop.DesktopLauncher`
<br>*Working directiory:* `/Users/missionbit/Desktop/Projects/FlappyDemo/android/assets`
<br>*Use classpath of module:* Select `desktop`
6. Click `Apply` and then `OK`.


## Game States and the GameStateManager
1. Navigate to `core/src/com.missionbit.game` >> Right-click (control + click) and select `New` > `Package`. 
<br>*Name:* `states` >> OK 
> *Hint:* Don't forget that we use lowercase for package names

2. Right-click the newly created states package and select `New` > `Java Class`
<br>*Name:* `State` >> OK 
> *Hint:* Don't forget that we capitalize class names

3. Make the State class abstract
4. Each state needs: a **camera** to locate a position in the game world, a **mouse** or some sort of pointer to indicate where the user clicks, which is defined as a Vector3, ie, a (x,y) coordinate, and a GameStateManager to manage our state transitions (for example, from menu to play, or from play to pause. 
<br>So we must declare the following State instance variables: 
```java
protected OrthographicCamera cam;
protected Vector3 mouse;
protected GameStateManager gsm;
```
5. Create a protected State constructor that takes in a GameStateManager and initializes all instance variables.
6. Create the following methods:
```java
protected abstract void handleInput();
public abstract void update(float dt);
public abstract void render(SpriteBatch sb);
public abstract void dispose();
```
> *Hint:* It’s important to dispose of any Texture or other media when we’re done using them to prevent any kind of memory leaks.

7. Navigate to the FlappyBird class, and declare the following instance variables:
```java
public static final int WIDTH = 480;
public static final int HEIGHT = 800;
public static final String TITLE = "Flappy Bird";
private GameStateManager gsm;
private SpriteBatch batch;
```
> *Hint:* We should only have one SpriteBatch in the entire game. They are pretty heavy.

8. Now navigate to `desktop/src/com.missionbit.game.desktop`, open `DesktopLauncher`, and insert these 3 lines inside the main method, right in the middle, in between the two lines of code (one that declares and initializes config, and the other that instantiates a `LwjglApplication`):
```java
config.width = FlappyDemo.WIDTH;
config.height = FlappyDemo.HEIGHT;
config.title = FlappyDemo.TITLE;
```

9. Create a new Java class inside of the states package named `GameStateManager` with the following code:
```java
public class GameStateManager {
   private Stack<State> states;
 
   public GameStateManager() {
       states = new Stack<State>();
   }
 
   public void push(State state) {
       states.push(state);
   }
 
   public void pop() {
       states.pop();
   }
 
   public void set(State state) {
       states.pop();
       states.push(state);
   }
 
   public void update(float dt) {
       states.peek().update(dt);
   }
   
   public void render(SpriteBatch sb) {
       states.peek().render(sb);
   }
}
```

10. Create a new Java class inside of the states package named `MenuState`, which extends `State`.
11. Right-click MenuState and select `Generate` > `Implement Methods…`
12. Select `MenuState`, `handleInput`, `update`, `render`, and `dispose` and click OK. Then make sure all methods are public.
> *Hint:* If there are any class names in red, option+click them to import the necessary packages.

13. In the `FlappyDemo` class, initialize and push the GameStateManager inside the `create()` method. Your code should look like this:
```java
  @Override
  public void create () {
     batch = new SpriteBatch();
     gsm = new GameStateManager();
     gsm.push(new MenuState(gsm));
     Gdx.gl.glClearColor(0, 0, 0, 1);
  }
```
14. In the `render()` method, update and render your `gsm`. Your code should look like this:
```java
  @Override
  public void render () {
     Gdx.gl.glClear(GL20.GL_COLOR_BUFFER_BIT);
     gsm.update(Gdx.graphics.getDeltaTime());
     gsm.render(batch);
  }
```
15. Finally, make sure to dispose of your batch:
```java
  @Override
  public void dispose () {
     batch.dispose();
  }
```
16. Run (you should see a black screen)

## Menu - Importing images
1. Go to https://github.com/BrentAureli/FlappyDemo and download the repo. Unzip it and copy the images from android > assets to the android > assets in your project
2. Inside `MenuState`, create the instance variables:
```java
private Texture background;
private Texture playBtn;
```

3. Option+click on Texture to import class. In the constructor, initialize them to: 

```java
background = new Texture("bg.png");
playBtn = new Texture("playBtn.png");
```

4. Stil in MenuState, inside of the render method, create a batch to draw the background and play button images:
```java
sb.begin();
sb.draw(background, 0, 0, FlappyDemo.WIDTH, FlappyDemo.HEIGHT);
sb.draw(playBtn, (FlappyDemo.WIDTH / 2) - (playBtn.getWidth() / 2), FlappyDemo.HEIGHT / 2);
sb.end();
```

> *Hint:* Think of the sprite batch as a container, it needs to be opened and closed.
          <br>Also, notice the code for centering the play button. If we just divide it by 2, the bottom left corner of the button would be at the center, so we need to offset it by half of its width.

5. Inside the dispose method, dispose of the background and play button:
```java
background.dispose();
playBtn.dispose();
```
6. Run.



## The Play State

1. Create a new Java class named "PlayState" inside the package states.
2. PlayState extends State
3. Right-click the class name and select Generate > Implement methods...
4. Right-click the class name and select Generate > Constructor
5. In the PlayState class, create an instance variable named bird of type Texture and initialize it with "bird.png", and draw the bird at position (50,50)
6. In the MenuState class, the update method checks if user did something by calling the handleInput method
7. The handleInput uses Gdx.input.justTouched() to check for user input, and, if the user did something, it calls gsm.set(new PlayState(gsm)) and dispose()
8. Run. Now the button should be clickable and, when clicked, the little bird should appear at the bottom left corner. Notice that the bird appears very small. We can use our camera to set a viewport, so we see only a small portion of our game world. It'll appear as if the image was zoomed in.
#### Cameras and Viewports
9. In our PlayState constructor, set our orthographic camera (which was created in State) to the bottom left quarter of the screen, with the y-axis pointing up.
>*Hint*: the libGDX OrthographicCamera has a method setToOrtho that takes 3 parameters: (boolean yDown, float viewportWidth, float viewportHeight). This "sets this camera to an orthographic projection, centered at (viewportWidth/2, viewportHeight/2), with the y-axis pointing up or down."
--[Check out the documentation](https://libgdx.badlogicgames.com/nightlies/docs/api/com/badlogic/gdx/graphics/OrthographicCamera.html#setToOrtho-boolean-float-float-)

10. In our PlayState render method, tell the SpriteBatch to set the projection matrix to the camera viewport
>*Hint*: the libGDX SpriteBatch has a setProjectionMatrix method that takes in a Matrix4 projection and "Sets the projection matrix to be used by this Batch"
--[Check out the documentation](https://libgdx.badlogicgames.com/nightlies/docs/api/com/badlogic/gdx/graphics/g2d/SpriteBatch.html#setProjectionMatrix-com.badlogic.gdx.math.Matrix4-)
<br>*Hint 2*: the libGDX OrthographicCamera has an instance variable "combined" of the type Matrix4, which represents "the combined projection and view matrix"
--[Check out the documentation](https://libgdx.badlogicgames.com/nightlies/docs/api/com/badlogic/gdx/graphics/Camera.html#combined)

11. Run. You should see the screen zoomed in on the bottom left quarter


## The Bird Class

1. Create a new package called sprites, and inside it a new Java class named "Bird"
2. Our Bird class needs the following attributes:
* a position, to determine where the bird is in the game world
* a texture, to determine the visual representation of the bird, ie, what will be drawn on the screen
* a velocity, to determine in which direction the bird moves (up, down, left, right)
* a gravity, to determine the acceleration pulling the bird down (so he falls when not flying)
Declare these instance variables as:
```java
private Vector3 position;
private Vector3 velocity;
private Texture bird;
private static final int GRAVITY = -15;
```

3. Setup the Bird constructor, which takes a x and a y as integers:
```java
public Bird(int x, int y){
    position = new Vector3(x, y, 0);
    velocity = new Vector3(0, 0, 0);
    bird = new Texture("bird.png");
}
```

4. Create a method update to recalculate the bird's position in our game world. Note that we need to scale the velocity by the change in time (represented by `dt`) before adding it to our position.
```java
public void update(float dt){
    velocity.add(0, GRAVITY, 0);
    velocity.scl(dt);
    position.add(0, velocity.y, 0);
    velocity.scl(1 / dt);
}
```

5. Right-click on the class name (`Bird`) and `Generate...` > `Getter` > Select `position` and `bird` > Click `OK`. Refactor `getBird()` to `getTexture()`.

6. Navigate to the `PlayState` class and remove the bird Texture. 
* Recreate bird as an instance variable of the type Bird.
* Initialize bird in the position (50, 100).
* Inside the `update()` method, handle user input and update the bird's position.
```java
@Override
public void update(float dt) {
    handleInput();
    bird.update(dt);
}
```

7. Change the render method so it draws the updated bird.
```java
sb.draw(bird.getTexture(), bird.getPosition().x, bird.getPosition().y);
```

8. Run. You should see the bird falling down. Try changing its initial coordinates to see it falling from different areas on the screen.


## Make Birds Fly

1. Inside of the Bird class, create a public void method called `jump`. This method sets the `velocity.y` to 250.

2. In the PlayState class, use the `handleInput` method to make the bird jump whenever the user clicks the screen.
>*Hint:* Use `Gdx.input.justTouched()` to check for user input.

3. Still in the PlayState, create a private instance variable for the background and call it bg. Initialize it to `bg.png`.

4. In the render method, draw the background. 
>*Hint:* The coordinates for the background are ((cam.position.x - cam.viewportWidth / 2), 0). Why?

5. Test it.

6. Make it so the game stops if the bird hits the ground (aka as the bottom of the screen). To do that, edit the method update. When the `position.y` goes below zero, set it to zero (remember if statements). In addition, only add `GRAVITY` if the bird is not already at zero.

7. Test it.


## Create the Tubes

1. Inside the sprites package, create a class Tube.

2. Tube needs two Textures, topTube and bottomTube. 
>*Hint:* Instance variables should be private.

3. Create a constructor that takes in a `float x` indicating the position where the tube should start. 
>*Hint:* Take a look at our image assets and figure out which one we should use to initialize our tubes.

4. Tube needs a few more instance variables:
```java
private static final int TUBE_GAP = 100; //opening between tubes
private static final int LOWEST_OPENING = 120; //lowest position the top of the bottom tube can be, must be above 90 to be above ground level
private static final int FLUCTUATION = 130; //may adjust to keep top tube in view
private Vector2 posTopTube, posBottomTube;
private Random rand;
```

5. Initialize the tubes position using Random.
```java
rand = new Random();
posTopTube = new Vector2(x, rand.nextInt(FLUCTUATION) + LOWEST_OPENING + TUBE_GAP);
posBottomTube = new Vector2(x, posTopTube.y - TUBE_GAP - bottomTube.getHeight());
```

6. Generate Getters for top/bottom tubes and their respective positions.

7. Inside the PlayState, create a tube and initialize its position on the x-axis to 100.

8. Draw the tubes.
>*Hint:* Take a look at how we drew the bird.

9. Run it multiple times. What happens to the tube position?


## Flying through obstacles

1. In the Tube class, create a `reposition` method that takes in a `float x` and *sets* the positions of the top and bottom tubes.
>*Hint:* You can use the same positions used to initialize posTopTube and posBotTube since they were positioned randomly.
<br>*Hint:* Vector2 has a `set` method.

2. In the PlayState class, create the following constants:
```java
private static final int TUBE_SPACING = 125;
private static final int TUBE_COUNT = 4;
```

3. Replace the tube in the PlayState class with an array of Tubes (`Array<Tube> tubes`).

4. Initialize the array and add `TUBE_COUNT` tubes using `tubes.add(new Tube(i * (TUBE_SPACING + Tube.TUBE_WIDTH)));`

5. In the Tube class, create a public constant for the TUBE_WIDTH and set it to 52.

6. In the PlayState class, inside the update method, create the logic to reposition tubes when they get out of the camera viewport. 
```java
if(cam.position.x - cam.viewportWidth / 2 > tube.getPosTopTube().x + tube.getTopTube().getWidth()){
    tube.reposition(tube.getPosTopTube().x +((Tube.TUBE_WIDTH + TUBE_SPACING) * TUBE_COUNT));
}
```

7. In the Bird class, create a constant `MOVEMENT` and set it to 100. This represents the horizontal movement of the bird.

8. In the update method, make sure the position on the x axis is also updated by passing in `MOVEMENT * dt` as a paramenter to the x argument.

9. In the PlayState class, update the camera's position in the game world based on the position of the bird. To do that, inside the update method set the `cam.position.x` equal to `bird.getPosition().x + 80`.

10. At the end of the update method, tell libGDX that the camera has been repositioned by calling `cam.update();`.


## Collision Detection

1. To handle collisions, we'll draw rectangles around the tubes. In the class `Tube`, declare two objects of type `Rectangle`: `boundsTop` and `boundsBot`.

2. Initialize them in the constructor.
>*Hint:* The constructor for Rectangle takes in 4 parameters: the x coordinate, the y coordinate, the width, and the height. Since we want these rectangles to completely cover the tubes, how can we use instance variables and attributes from the Tube class to pass in the desired parameters for each rectangle?

3. Remember to also reposition the rectangles in the `reposition` method.
>*Hint:* Rectangle has a `setPosition(float x, float y)` method.

4. Create a `collides` method that takes in a `Rectangle player` and returns true if the player overlaps the `boundsTop` or `boundsBot`, but false otherwise.

5. Now we need to create a `Rectangle bounds` for the bird. Don't forget to initialize it.

6. Every time the bird moves we need to update its bounds. Use the `setPosition(float x, float y)` to update the bounds whenever the bird moves. 

7. Create a getter method for `bounds`.

8. In the `PlayState` class, we already have a `for loop` that cycles through our `tubes` inside the `update` method. We can use that same `for loop` to check if the `bird` collides with any `tube`. If there is a collision, call `gsm.set(new PlayState(gsm))` and `break` out of the loop.


## Dispose Resources

1. In the `GameStateManager`, every time we `pop` a state we should `dispose` of it since we can't reuse states. Note that `states.pop()` is called twice in this class.

2. In the `MenuState` class, remove the call to `dispose` from the `handleInput` method because now we're disposing of states when we `pop` them off. What happens when the method `dispose` from the `MenuState` class gets called? 

3. In the `PlayState` class, we need to dispose of `bg`, `bird`, and all of the `tubes`. Note that you'll have to implement the `dispose` methods for the `Bird` and `Tube` classes.

4. Add log messages to the `dispose` methods in PlayState and MenuState.

5. Run. You should see the log messages every times the game restarts.


## Placing the Ground

1. The ground will be created the same way as the tubes: it'll be declared as a Texture that gets repositioned every time it goes off of the screen. Create `ground` as an instance variable of the class `PlayState`, and initialize it using `ground.png`.

2. Create `groundPos1` and `groundPos2` as instance variables of type `Vector2`. Initialize `groundPos1` to the positions of the bottom left corner of our camera `cam`. Initialize `groundPos2` to the position directly to the right of `groundPos1`.

3. Draw the `ground` twice, at positions `groundPos1` and `groundPos2`.

4. Run. What do you notice?

5. Create a constant `GROUND_Y_OFFSET` that equals to -50, and change the initial y value of `groundPos1` and `groundPos2` to this constant.

6. Create a method `updateGround()` that checks if each of the ground right edges is to the left of the left edge of the viewport, ie, it checks if each of the ground projections are out of the camera's field of vision. If it is, use the method `add(float x, float y)` from the `Vector2` class to update the ground position, ie, `groundPos1` or `groundPos2`.

7. Make sure to call `updateGround()` in the `update` method.

8. Kill the bird if it hits the ground. You can do that by checking if the bird's position on the y axis is less than or equal to the y position of the top edge of the ground.
>*Hint:* The top edge of the ground is the ground's height + offset.
<br>*Hint:* To kill the bird, we do the same as we did when the bird collides with a tube: restart the game.

9. Make sure to `dispose` of the ground Texture.

10. Try it out!


## Animating Sprites

1. Inside of the `sprites` package, create a new class `Animation`.

2. The `Animation` class needs the following attributes:
```java
Array<TextureRegion> frames; //where we store all of our frames
float maxFrameTime; //this determines how long a frame needs to stay in view before switching to the next one
float currentFrameTime; //how long the animation has been in the current frame
int frameCount; //number of frames in our animation
int frame; //the current frame we're in
```

3. Now we need to initialize these instance variables in the constructor:
```java
public Animation(TextureRegion region, int frameCount, float cycleTime){
    frames = new Array<TextureRegion>();
    TextureRegion temp;
    int frameWidth = region.getRegionWidth() / frameCount;
    for(int i = 0; i < frameCount; i++){
        temp = new TextureRegion(region, i * frameWidth, 0, frameWidth, region.getRegionHeight());
        frames.add(temp);
    }
    this.frameCount = frameCount;
    maxFrameTime = cycleTime / frameCount;
    frame = 0;
}
```

4. We also need an `update` method to cycle through the frames.
```java
public void update(float dt){
    currentFrameTime += dt;
    if(currentFrameTime > maxFrameTime){
        frame++;
        currentFrameTime = 0;
    }
    if(frame >= frameCount)
        frame = 0;

}
```

5. Finally, we need a getter for the frames.
```java
public TextureRegion getFrame(){
    return frames.get(frame);
}
```

6. In the `Bird` class, we need to create a `birdAnimation`, which will be initialized with `birdanimation.png`, a frame count of 3, and a cycle time of 0.5f.

7. Now that we have a `birdAnimation`, we need to get rid of `bird` and replace it with the new `texture`, but remember this one is 3 times as wide.

8. In `update`, update the `birdAnimation` and don't forget to pass in the delta time.

9. In `getTexture`, now we need to return `birdAnimation.getFrame()` instead.

10. Also update the `dispose` method to dispose of the new `texture` instead of `bird`.


## Music & Sound Effects

1. Find the music and sound effect files in our assets folder.

2. Inside `FlappyDemo`, create a `music` instance variable of the type `Music` and initialize it to `Gdx.audio.newMusic(Gdx.files.internal("music.mp3"))`

3. inside the constructor, set the configurations for `music` as follows:
```java
music.setLooping(true);
music.setVolume(0.1f);
music.play();
```

4. Create a dispose method (Generate > Override) and dispose of `music`.

5. To create the sound effect, go to the `Bird` class and create an instance variable `Sound flap`, which will be initialized to `Gdx.audio.newSound(Gdx.files.internal("sfx_wing.ogg"))`

6. Inside the `jump` method, call `flap.play()`. To change the volume of the sound effect, pass in a float, say, 0.5f.

7. Dispose of `flap`.


## Porting to Android

1. Go to android > AndroidManifest.xml and change the screenOrientation to portrait.

2. Copy the `cam.setToOrtho...` from the `PlayState` to the `MenuState`.

3. Copy the `sb.setProjectionMatrix...` from the `PlayState` to the `MenuState`.

4. Update the following lines in the `MenuState`:
```java
sb.draw(background, 0,0);
sb.draw(playBtn, cam.position.x - playBtn.getWidth() / 2, cam.position.y);
```
