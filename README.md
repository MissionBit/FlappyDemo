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
16. Run (you should see a red screen)

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
    texture = new Texture("bird.png");
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











