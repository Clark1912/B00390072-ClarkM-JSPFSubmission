# Documentation
This documentation will outline multiple areas in relation to element 5.

## Multiple Scenes
Element 5 features a menu scene and a game scene that the user will transition between. The menu scene is comprised of a skybox and a play button; it also creates the background music. The game scene features a skybox, a menu button, a score counter, a ground plane, a player character, a box, and fences.

### Menu Scene
The menu scene allows the player to click play and start the game. When the player presses the play button it will play a sound and transition to the game scene.
```Play Button Code
// Create button
function createSceneButton(scene: Scene, name: string, index: string, x: string, y: string, advtex) {
    let button = GUI.Button.CreateSimpleButton(name, index);
      button.left = x;
      button.top = y;
      button.width = "160px";
      button.height = "60px";
      button.color = "white";
      button.cornerRadius = 20;
      button.background = "green";

    // Audio
    const buttonClick = new Sound("MenuClickSFX", "./audio/menu-click.wav", scene, null, {
        loop: false,
        autoplay: false,
    });

    // Button clicked
    button.onPointerUpObservable.add(function() {
          buttonClick.play();
          setSceneIndex(1);
    });
    advtex.addControl(button);
    return button;
}
```

The background music is set up in a way that it is present in both scenes. This code can be seen below:
```Background music code
// Play background music
const backgroundMusic = new Sound("Background Music", "Audio/soft-piano-100-bpm-121529.mp3", that.scene, null, { loop: true, autoplay: true});
```

### Game Scene
The game scene contains all the gameplay. If the player character collides with the crate, the score counter will increase by 1. The score counter was created by including a GUI textblock as can be seen below:
```Textblock code
// Create textblock
function createTextblock(scene: Scene, text: string, advancedTexture, name: string, posTop: number, posLeft: number)
{
    let textblock = new GUI.TextBlock(name, text);
    textblock.color = "white";
    textblock.fontSize = 20;
    textblock.top = posTop + "%";
    textblock.left = posLeft + "%";

    advancedTexture.addControl(textblock);

    return textblock;
}
```
When the menu button is pressed, the scene will change to the menu scene; the menu button acts the same as the play button does for the menu scene except it changes the scene from the game scene to the menu scene.

The background music continues from the menu scene without the need for it to be declared again.

When this scene is rendered, as well as initialising the UI, it also creates the objects that are required for the game and initialises the physics.
```Creation of Objects and Initialisation of Physics Code
export default function GameScene(engine: Engine) {
    interface SceneData {
      scene: Scene;
      ground?: Mesh;
      fence?: any;
      box?: Mesh;
      importPlayerMesh?: any;
      actionManager?: any;
      skybox?: Mesh;
      light?: Light;
      camera?: Camera;
    }
  
    let that: SceneData = { scene: new Scene(engine) };

    // Initialise physics
    that.scene.enablePhysics(new Vector3(0, -9.8, 0), havokPlugin);

    that.ground = createGround(that.scene);

    boxX = Math.random() * 10;
    boxZ = Math.random() * 10;

    that.box = createBox(that.scene, boxX, 5 ,boxZ);

    that.importPlayerMesh = importPlayerMesh(that.scene, that.box);
    that.actionManager = actionManager(that.scene);

    that.fence = createFences(that.scene);

    that.skybox = createSkybox(that.scene);
    that.light = createLight(that.scene);
    that.camera = createArcRotateCamera(that.scene);

    // GUI
    let advancedTexture = GUI.AdvancedDynamicTexture.CreateFullscreenUI("myUI", true);
    let scoreText = createTextblock(that.scene, "Score: " + score.toString(), advancedTexture, "scoreText", -45, -45);
    let menuButton = createMenuButton(that.scene, "button1", "Menu", "40%", "-40%", advancedTexture);

    ...

    return that;
  }
```

## index.ts
The index.ts file allows for the scenes to be changed. It does this storing the scenes in an array and executing a function that will render the correct scene based on the value passed into the function. The function accepts a number, this number corresponds to the position in the array of the desired scene. The function can be seen below.
```setSceneIndex() function
export default function setSceneIndex(i: number)
{
    eng.runRenderLoop(() => {
        scenes[i].scene.render();
    })
}
```

The setSceneIndex function has to be imported into both scene typescript files so that the scenes can be changed by the buttons.
```Import index.ts Code
import setSceneIndex from "./index";
```

## Skybox
A skybox is used in the menu scene and the game scenes so to provide the illusion of a vast world surrounding the player. The skybox works by applying different textures to each face of a cube that line up so that the seams cannot be seen.
```Skybox Code
// Create skybox
function createSkybox(scene: Scene)
{
    //Skybox
    const skybox = MeshBuilder.CreateBox("skyBox", {size:150}, scene);
    const skyboxMaterial = new StandardMaterial("skyBox", scene);
    skyboxMaterial.backFaceCulling = false;
    skyboxMaterial.reflectionTexture = new CubeTexture("textures/skybox4", scene);
    skyboxMaterial.reflectionTexture.coordinatesMode = Texture.SKYBOX_MODE;
    skyboxMaterial.diffuseColor = new Color3(0, 0, 0);
    skyboxMaterial.specularColor = new Color3(0, 0, 0);
    skybox.material = skyboxMaterial;

    return skybox;
}
```
The skybox that was chosen for this element was found in the Texture Library by babylon.js.

## Physics
The Havok physics engine is used in this element to provide physics to the objects. The physics system applies gravity and allows for collisions to take place. Havok needs to be imported and initialised at the start of the document and then initialised in the game scene function that allows the scene to be rendered.

```Import and initialisation of physics
// Initialisation of physics
let initializedHavok;
HavokPhysics().then((havok) => {
    initializedHavok = havok;
});

const havokInstance = await HavokPhysics();
const havokPlugin = new HavokPlugin(true, havokInstance);

globalThis.HK = await HavokPhysics();
```

```Physics in the game scene function
// Initialise physics
that.scene.enablePhysics(new Vector3(0, -9.8, 0), havokPlugin);
```

## Collision Detection
The collision detection between the player and the crate allows for the score to be incremented. The collision is a process on the player rather than on the box. The code can be seen below:

``` Code
// Collision
    if (mesh.intersectsMesh(collider)){
        score++;
        // Destroy box
        collider.dispose();

        // Randomise positions
        if (Math.round(Math.random()) == 0)
        {
            boxX = Math.random() * 10;
        }
        else
        {
            boxX = Math.random() * -10;
        }

        if (Math.round(Math.random()) == 0)
        {
            boxZ = Math.random() * 10;
        }
        else
        {
            boxZ = Math.random() * -10;
        }

        // Create a new box
        collider = createBox(scene, boxX, 5, boxZ);
    }
```
The if statement checks if the player mesh intersects with the collider (crate) mesh. It will then increment the score. Due to issues with changing the position of the box in regards to the physics that had been applied to the box, the box needed to be destroyed so that it could be recreated with a new position. The new position is randomly generated. As the Math.random() function only returns values between 0 and 1 and did not take parameters to specify the range of the number that should be generated, another method had to be applied. To randomise the number, a different random number was generated and rounded so that it was either 0 or 1. If the value was 0 it would randomise a positive number for the X position of the box to spawn in. If it was a 1, then a negative number would be generated for the box's X position. The same process is performed to find the value of the box's new Z position.

## Imported Meshes
### Player
The player model is from the Meshes Library that babylon.js provide. It includes animations. These animations are triggered by the player moving. When the player presses either the W, A, S, D, or arrow keys, it moves the player and sets the keydown variable to true so that an animation can play. This code is included in the scene.onBeforeRenderObservable.add() function which means that the code in that function is triggered before rendering the scene and will allow for the code to be executed at any point in the scene.

```scene.onBeforeRenderObservable.add() function
scene.onBeforeRenderObservable.add(()=> {

    let keydown: boolean = false;

    if (keyDownMap["w"] || keyDownMap["ArrowUp"]) {
        mesh.position.z += 0.1;
        mesh.rotation.y = 0;

        keydown = true;
    }
    if (keyDownMap["a"] || keyDownMap["ArrowLeft"]) {
        mesh.position.x -= 0.1;
        mesh.rotation.y = 3 * Math.PI / 2;

        keydown = true;
    }
    if (keyDownMap["s"] || keyDownMap["ArrowDown"]) {
        mesh.position.z -= 0.1;
        mesh.rotation.y = 2 * Math.PI / 2;

        keydown = true;
    }
    if (keyDownMap["d"] || keyDownMap["ArrowRight"]) {
        mesh.position.x += 0.1;
        mesh.rotation.y = Math.PI / 2;
        
        keydown = true;
    }

    if (keydown) {
        if (!animating) {
        animating = true;
        scene.beginAnimation(skeleton, walkRange.from, walkRange.to, true);
        }
    }
    else {
        animating = false;
        scene.stopAnimation(skeleton);
    } 
```

### Fences
The fences were found in the Babylon.js Meshes Library. The fences were added to allow the player to see where the crate can spawn. Due to the file for the fence being a .gltf file, an additional import had to be added to the code so that the model could be imported. The code for this import can be seen below:
```Code
import "@babylonjs/loaders/glTF";
```
As the fence is shorter than the width of the ground it is supposed to surround, multiple instances of it had to be created. The creation of the instances takes place in a function that is executed when the fence mesh is imported.
```Code
let fence: any = SceneLoader.ImportMesh("", "./models/", "fenceASection1.gltf", scene, function(meshes){
    let mesh = meshes[1] as Mesh;

    mesh.isVisible = false;
```
It was important to set the mesh to the second item in the meshes array as setting it as the first item caused issues. It was also important to assign the mesh variable "as Mesh" as without it, the mesh would be assigned as an AbstractMesh which is not usable with the createInstance() function. Setting the mesh to invisible by changing its isVisible attribute to false allowed for the instances to be shown but not the root object which was not in a correct position.

An array of places that the fences should spawn at was created with the first value storing the rotation of the fence (in radians), the second storing the X position of the fence, and the final value being the Z position of the fence.
```Places array
// Front positions
const places: number[] [] = [];
places.push([0, 10, 11.25]);
places.push([0, 7.5, 11.25]);
places.push([0, 5, 11.25]);
places.push([0, 2.5, 11.25]);
places.push([0, 0, 11.25]);
places.push([0, -2.5, 11.25]);
places.push([0, -5, 11.25]);
places.push([0, -7.5, 11.25]);
places.push([0, -10, 11.25]);
// Right positions
places.push([1.5708, 11.25, 10]);
places.push([1.5708, 11.25, 7.5]);
places.push([1.5708, 11.25, 5]);
places.push([1.5708, 11.25, 2.5]);
places.push([1.5708, 11.25, 0]);
places.push([1.5708, 11.25, -2.5]);
places.push([1.5708, 11.25, -5]);
places.push([1.5708, 11.25, -7.5]);
places.push([1.5708, 11.25, -10]);
// Back positions
places.push([6.28319, 10, -11.25]);
places.push([6.28319, 7.5, -11.25]);
places.push([6.28319, 5, -11.25]);
places.push([6.28319, 2.5, -11.25]);
places.push([6.28319, 0, -11.25]);
places.push([6.28319, -2.5, -11.25]);
places.push([6.28319, -5, -11.25]);
places.push([6.28319, -7.5, -11.25]);
places.push([6.28319, -10, -11.25]);
// Left positions
places.push([-1.5708, -11.25, 10]);
places.push([-1.5708, -11.25, 7.5]);
places.push([-1.5708, -11.25, 5]);
places.push([-1.5708, -11.25, 2.5]);
places.push([-1.5708, -11.25, 0]);
places.push([-1.5708, -11.25, -2.5]);
places.push([-1.5708, -11.25, -5]);
places.push([-1.5708, -11.25, -7.5]);
places.push([-1.5708, -11.25, -10]);
```

A for loop then allowed for instances to be created for every place in the array. The fence instances were stored in an array, with each instance in the array having it's corresponding rotation, X position, and Z position assigned.
```Creation of instances
const fences: InstancedMesh[] = [];
for (let index = 0; index < places.length; index++)
{
    fences[index] = mesh.createInstance("fence" + index);
    fences[index].rotation = new Vector3(0, places[index][0], 0);

    fences[index].position.x = places[index][1];
    fences[index].position.z = places[index][2];
}

fence = fences;
});

return fence;
```
The variable fence was assigned the array of fence instances as it allows for the array to be returned and the instances to be created.

## Textures
Textures have been applied to multiple game objects in the scenes. The skybox is made up of textures on all faces of the cube to provide the illusion of a sky. The box has a texture on it to make it appear like a crate. The ground has a texture on it to make it look like a stone floor. An example of a texture being applied can be seen below:

```Crate code
function createBox(scene: Scene, x: number, y: number, z: number){
    const mat = new StandardMaterial("mat");
    const texture = new Texture("textures/crate.png");
    mat.diffuseTexture = texture;
    
    let box: Mesh = MeshBuilder.CreateBox("box", {});
    box.position.x = x;
    box.position.y = y;
    box.position.z = z;

    box.material = mat;

    const boxAggregate = new PhysicsAggregate(box, PhysicsShapeType.BOX, { mass: 1 }, scene);
  
    return box;
}
```

## Preventative Measures
### Box
The box has been coded in a way that its position is randomised. The numbers that the randomisation can reach should prevent the box from falling off the edge of the ground, however, measures have been put in place to make sure that in the event that the box does fall off the edge the game can still be played. If the box falls below -5 on the Y axis, it will be destroyed and a new box will be created to land on the ground. The code is executed in a method that continuously executes after the scene has been rendered; the code can be seen below:

```Respawn Box Code
that.scene.onAfterRenderObservable.add(() => {

    ...

    // Check if box exists
    if (that.box != null){
    // Check if the box has fallen off the edge
    if (that.box.position.y < -5)
    {
        // Destroy box
        that.box.dispose();

        // Create a new box
        that.box = createBox(that.scene, 0, 5, 0);
    }
    }
});
```

### Player
To prevent the player from falling off the edge of the ground, the player has been given no mass. This means that if the player does go over the edge they will just float instead of falling.

```Player Physics Code
let playerAggregate = new PhysicsAggregate(item, PhysicsShapeType.CAPSULE, { mass: 0 }, scene);
playerAggregate.body.disablePreStep = false;
```

## GUI
### Play Button
The play button is displayed in the centre of the menu screen. When the button is clicked it will take the player to the game scene. This button is discussed more in depth in the menu scene section of this documentation.

### Score Text
The score text is displayed in the top left of the game scene. It displays the player's current score. It is constantly being updated in the scene.onAfterRenderObservable.add() function so that it is always accurate.
```Update Score Code
that.scene.onAfterRenderObservable.add(() => {
    // Update score
    scoreText.text = "Score: " + score.toString();
```
### Menu Button
The menu button is found in the top right corner of the game scene. This button functions like the play button in the menu scene. The differences between the play button and the menu button are that they are different colours, in different positions, and when clicked they transition to different scenes; the menu button transitions to the menu scene.

## Problems
### Physics
An issue that occurred that caused significant delays to the creation of this element was attempting to move the box back onto the ground when it had fallen off. The issue was caused because the box had physics applied to it and when physics is applied to an object, the object's position cannot be easily changed by manipulating its position attribute. Attempts to solve this issue included, turning off the physics engine and restarting it when the position of the box has been changed, and pausing the physics and then moving the box. The problem with these solutions was that while it was easy to stop or pause the physics engine, attempts to start it again failed. The solution that was found for this issue was to completely destroy the box and then recreating it at the desired position.

### Importing meshes
Another issue that took up a lot of time was importing meshes. When importing the fence mesh, errors appeared. One of these errors related to the file being a .gltf file. This meant that an additional module was needed to be imported so that .gltf files could be rendered. Another error that occurred in relation to importing meshes was that meshes import as an AbstractMesh. This caused issues with creating an instance of the mesh as the function for creating an instance was only for objects of type Mesh. To overcome this issue, when assigning a mesh from the meshes imported, "as Mesh" had to be included in the assignment.
```Assigning Mesh
let fence: any = SceneLoader.ImportMesh("", "./models/", "fenceASection1.gltf", scene, function(meshes){
      let mesh = meshes[1] as Mesh;
```
The mesh for the fence also had to be set to the second item in the array as the first item caused only one fence to spawn.

## Assets Used
Multiple assets were sourced for this element. Where they were found and who they were by can be seen below as well as links to the assets.

* Background Music - [Soft Piano - 100 BPM by royalty_free_music](https://pixabay.com/sound-effects/soft-piano-100-bpm-121529/)

* Skybox - [Skybox4 from The Texture Library from Babylon.js](https://github.com/BabylonJS/Babylon.js/tree/master/packages/tools/playground/public/textures)

* Crate Texture - [Crate from The Texture Library from Babylon.js](https://github.com/BabylonJS/Babylon.js/blob/master/packages/tools/playground/public/textures/crate.png)

* Ground Texture - [Floor from The Texture Library from Babylon.js](https://github.com/BabylonJS/Babylon.js/blob/master/packages/tools/playground/public/textures/floor.png)

* Fence - [FenceASection1 from The Meshes Library from Babylon.js](https://github.com/BabylonJS/Assets/tree/master/meshes/graveYardPack/fenceASection1/gltf)

* Player Character - [Dummy3 from The Meshes Library from Babylon.js](https://github.com/BabylonJS/Assets/blob/master/meshes/dummy3.babylon)

* Button Click Sound - Lab Notes