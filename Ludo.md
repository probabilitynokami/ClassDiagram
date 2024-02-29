# Ludo Class Diagram

## Class Diagram

```mermaid
classDiagram
    namespace GameFramework{

        class ISceneManager{
            <<interface>>
            + void StageSceneExit()
            + void StageSceneInsert(IScene s)
            + void CommitScene()
            + void GetCurrentScene()
        }
        class SceneManagementCommand{
            <<enumeration>>
            INSERT
            EXIT
        }
        class GameEngine{
            # Stack~IScene~ _sceneStack
            # Queue~SceneManagementCommand~ sceneCommand
            # Queue~IScene~ stagedScene
            + void Run()
            # void Loop()
            # void Render()
        }
        class IScene{
            <<interface>>
            + void Update()
        }

        class IRenderable{
            <<interface>>
            + void Draw()
        }
        class ConsoleRenderable{
            + ConsoleRenderable()
        }
        class RenderSystem_T_{
            <<static>>
            +List~T~ Renderables
            +RegisterRenderable(T)
        }
        class RenderSystem_ConsoleRenderable_{
        }

        class ConsoleGameEngine{
        }
    }




    namespace GameObject{
        class Player{
            <<interface>>
            + ID : readonly
        }
        class PlayerWithAction{
            <<interface>>
            + ID : readonly
            + IActionable GetActionable()
        }
        class IActionable{
            <<interface>>
            + Step()
        }

        class IContextManager_T_{
            <<interface>>
            + T GetContext()
        }

    }

    namespace LudoGame{
        class IContextManager_LudoContext_{
            <<interface>>
        }

        class LudoContext{   
            + List~PlayerWithAction~ players
            + Board board
            - Dictionary~ Player,List ~Totem~ ~ _playerTotem
            + List~Totem~ GetTotems(Player)
        }

        class LudoGameScene{
            - ISceneManager _sceneManager
            - LudoContext _context
            - void NextTurn()
        }

        class LudoRule{
            - IContextManager _contextManager
            - Func<Board,bool> _ruleSet
            + bool Check(IActionable)
            + bool statusCheck()
            + void RegisterRule(Func~Board,bool~)
        }
        class LudoActionable{
            - int power
        }
        class SingleTotemActionable{
            - Totem bindedTotem
            * void Bind(totem)
        }
        class LudoTotemStart{
        }
        class LudoTotemMove{
        }
        class LudoTotemMoveTogether{
            - Totem
            + void Bind(Totem, Totem)
        }


    }


    namespace Utility{
        class Path{
            + List~int~ Path
        }
    }
    

    namespace LudoObjects{



        class Board{
            +List~Cell~ Cells : readonly
            +List~Path~ Paths : readonly

        }

        class Cell{
            + CellType type : readonly
            - List~Totem~ Occupants
            + AddTotem(Totem)
            + KickTotem(Totem)
            + GetOwnership()
        }

        class CellType{
            <<enumeration>>
            Normal
            Safe
        }

        class Totem{
            + mathvector position : public get
            + mathvector homePosition : public get
            - List~int~ path
            + IPlayer Owner : readonly
            + AdvanceOnce()
            + GoHome()
        }
        class LudoDice{
            +GetLastRoll()
            +Roll()
        }
    }

    namespace LudoObjectsRendering{
        class TotemRendering{
            - Totem totem
        }
        class BoardRendering{
            - Board board
        }
        class LudoDiceRendering{
            - LudoDice dice
        }
    }

    %% LudoObjectsRendering relationships
    TotemRendering *-- Totem
    BoardRendering *-- Board
    LudoDiceRendering *-- LudoDice
    BoardRendering --|> ConsoleRenderable
    TotemRendering --|> ConsoleRenderable
    LudoDiceRendering --|> ConsoleRenderable

    %% GameFramework relationships
    ISceneManager -- IScene
    GameEngine --|> ISceneManager
    GameEngine o-- IScene
    ConsoleRenderable --|> IRenderable
    RenderSystem_ConsoleRenderable_ ..|> RenderSystem_T_ : bind T as ConsoleRenderable
    RenderSystem_ConsoleRenderable_ -- ConsoleRenderable

    ConsoleGameEngine --|> GameEngine
    ConsoleGameEngine -- RenderSystem_ConsoleRenderable_

    %% LudoObjects relationship
    Cell --* Board
    Cell -- CellType
    LudoContext *-- Board
    LudoContext *-- PlayerWithAction 

    Totem -- LudoTotemMoveTogether
    Totem -- SingleTotemActionable

    Totem -- Cell

    %% LudoGame relationship
    IScene <|-- LudoGameScene
    IContextManager_T_ <|.. IContextManager_LudoContext_ : bind T as LudoContext
    LudoGameScene -- LudoContext
    LudoGameScene --|> IContextManager_LudoContext_


    LudoGameScene -- ISceneManager
    SingleTotemActionable <|-- LudoTotemStart
    SingleTotemActionable <|-- LudoTotemMove
    IActionable <|-- LudoActionable
    LudoActionable <|-- SingleTotemActionable
    LudoActionable <|-- LudoTotemMoveTogether
    LudoRule -- IActionable
    LudoGameScene -- LudoRule

    %% GameObjects relationships
    Player <|-- PlayerWithAction
    PlayerWithAction -- IActionable
```

## Rough Sequence Diagram

```mermaid
sequenceDiagram
    Main->>GameEngine: StageSceneInsert(mainMenu)
    Main->>GameEngine: CommitScene()
    Main->>GameEngine: Run()
    activate GameEngine
        GameEngine ->> MainMenuScene: Update()
        activate MainMenuScene
            MainMenuScene ->> GameEngine: StageSceneInsert(LudoGameScene)
            GameEngine ->> GameEngine: CommitScene()
        deactivate MainMenuScene

        GameEngine ->> LudoGameScene: Update()
        activate LudoGameScene
            LudoGameScene ->> LudoPlayer: GetActionable() 
            activate LudoPlayer
            LudoPlayer->>LudoGameScene : LudoActionable
            deactivate LudoPlayer
            LudoGameScene ->> LudoRule : Check(LudoActionable)
            activate LudoRule
                LudoRule ->> LudoActionable : step()
                LudoRule ->> LudoRule : CheckRule()
                LudoRule ->> LudoActionable : step()
                LudoRule ->> LudoRule : CheckRule()
                LudoRule -> LudoActionable : ...
                LudoRule ->> LudoGameScene : ok
            deactivate LudoRule

            LudoGameScene ->> LudoActionable : step()
            LudoGameScene ->> LudoActionable : step()
            LudoGameScene ->> LudoActionable : ...

            LudoGameScene ->> LudoGameScene : NextTurn()

            LudoGameScene ->> GameEngine : StageExitScene()
        deactivate LudoGameScene


    deactivate GameEngine

```

## Design Documentation

### GameFramework
The GameFramework namespace contains the classes to manage the game. It contains system for the game loop management, scene management, scene context, and render system.

#### GameEngine Class
The GameEngine class defines the way how the game is structured.

##### Run() and Loop()
Typically, implementation of the Run() method will run a setup code for the game and ten will call Loop() method which could run indefinitely.

```c#
public void Run(){
    // code to setup some things...
    this.Loop();
    // code to handle exits...
}
```

The following snippet is the expected implementation of Loop()
```c#
protected void Loop(){
    while(True){
        this.CommitScene(); // read ISceneManager
        this.GetCurrentScene().Update();
        this.Render();
    }
}
```
For game with physics, the time elapsed between the Loop call can be an important information. 
In this case, the time elapsed could be handled as an internal state of each scene, that is, each scene can measure the time difference by itself every time the Update() method is called.
If such methode doesn't fancy you, you can always derive the GameEngine class and implement a Loop() method with delta time.

##### Render()
The Render() method is for rendering the scene into a render media. 
Its implementation should iterate the static member Renderables of the RenderSystem class and call their Draw() method.
The Render() method could be called inside the Loop() method like the code snippet above, or it could also run inside a thread.
```c#
protected void Render(){
    foreach(IRenderable obj in RenderSystem<ConsoleRenderable>.Renderables){
        obj.Draw();
    }
}
```
#### IScene interface
IScene is the interface for each scene in a game.
A scene could be the main game, the menu, the pause menu, credit, and etc.
Each scene must implement Update() method which would be called in the GameEngine's Loop.

#### ISceneManager interface
This is an interface to enable scene management.
Each class that implement this interface must have a container for IScenes and SceneManagementCommand, and should pass its own reference to each IScene inside of that class.
The methods of this interface will manipulate the IScene's container.
This enable each IScene to control the flow of the program by providing a restricted interface to the GameEngine.

##### Staging methods
To make sure modification to IScene's container happen not while a scene's Update() method is running, the modification should be staged first and then applied at once in CommitScene() method.
This staging is done by pushing into a the SceneManagementCommand's container.
Below is an implementation example of the staging and commit.
```c#
public void StageSceneExit(){
    // using queue as command container
    this.sceneCommand.Enqueue(SceneManagementCommand.EXIT);
}
public void StageSceneInsert(IScene scene){
    this.sceneCommand.Enqueue(SceneManagementCommand.INSERT);
    // using queue to save staged scene, different from IScene container
    this.stagedScene.Enqueue(scene);
}
public void CommitScene(){
    while(this.sceneCommand.Count > 0){
        switch(this.sceneCommand.Peek()){
            case SceneManagementCommand.EXIT:
                this.sceneStack.Pop();
                break;
            case SceneManagementCommand.INSERT:
                this.sceneStack.Push(
                    this.stagedScene.Dequeue()
                );
                break;
        }
        this.sceneCommand.Dequeue();
    }
}
```

#### Rendering system
This design of the rendering system in this framework can be adapted to specific rendering device.
To do that, you must create a class for the rendering device.
In the class diagram, rendering on console is provided as an example.
When using console as rendering device, you can create a ConsoleRenderable class that implements the IDrawable interface.
Later, if an object want to be made renderable on console, it can simply inherit from ConsoleRenderable.
Additionally, in the constructor of ConsoleRenderable, or any class that implements IRenderable, must register themself to the RenderSystem class like shown in code below.
This way, all instances of ConsoleRenderable can be rendered like in GameEngine [Render() method](#render)

```c#
public ConsoleRenderable(){
    RenderSystem<ConsoleRenderable>.RegisterRenderable(this);
}
```

To decouple GameEngine class from the rendering device, you should derive the GameEngine class to a specific rendering device to override the Render() method. So, to use console as rendering device, a ConsoleGameEngine class like in the diagram should be made.
