# Ludo Class Diagram

## Old Class Diagram

```mermaid
classDiagram
    class Player{
        - Totem[] totem
        - Path path
        + Totem GetTotem(id)
        + GetPath()
        * Move()
    }
    class HumanPlayer{
        + DecideMove()
    }


    class Cell{
        - pair[float,float] _position
        - Enum _status
        + Totem[] Occupants : readonly
        + Cell(status, position)
        + GetPosition()
        + GetStatus()
        + PlaceTotem(Totem)
        + MoveOutTotem(Totem)
    }

    class Board{
        <<Data Class>>
        + readonly Cell[] cells
        + readonly int[][] paths
        + readonly int NumberOfPlayer
        + BuildFromConfig(filepath)
    }

    class Rule{
        * bool Check(Player, Cell)
    }
    class RuleBlocking{

    }
    class RuleOwnership{

    }

    class GameController{
        - Player[] players
        - int _currentPlayer
        - Board Board
        - Die die
        - Rule rule
        
        + GameController(int nPlayer, Board, Rule)
        + Run()
        - Loop()
        - NextTurn()

    }

    class Die{
        Roll()
    }


    RuleBlocking  --|> Rule
    RuleOwnership --|> Rule


    Player <|-- HumanPlayer

    GameController *-- Player
    GameController *-- Board
    GameController *-- Rule
    GameController *-- Die

    Board *-- Cell

```

## Better Ludo Class Diagram

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

        class GameEngine{
            # Stack~IScene~ _sceneQueue
            + void Run()
            # void Loop()
            # void Render()
        }
        class IScene{
            <<interface>>
            + void Update()
        }

        class IContextManager_T_{
            <<interface>>
            + T GetContext()
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

    ISceneManager -- IScene
    GameEngine --|> ISceneManager
    GameEngine o-- IScene
    ConsoleRenderable --|> IRenderable
    RenderSystem_ConsoleRenderable_ ..|> RenderSystem_T_ : bind T as ConsoleRenderable
    RenderSystem_ConsoleRenderable_ -- ConsoleRenderable

    ConsoleGameEngine --|> GameEngine
    ConsoleGameEngine -- RenderSystem_ConsoleRenderable_



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
    }
    Player <|-- PlayerWithAction
    PlayerWithAction -- IActionable

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
    LudoGameScene -- ISceneManager
    SingleTotemActionable <|-- LudoTotemStart
    SingleTotemActionable <|-- LudoTotemMove
    IActionable <|-- LudoActionable
    LudoActionable <|-- SingleTotemActionable
    LudoActionable <|-- LudoTotemMoveTogether
    LudoRule -- IActionable
    LudoGameScene -- LudoRule

    IScene <|-- LudoGameScene
    IContextManager_T_ <|.. IContextManager_LudoContext_ : bind T as LudoContext
    LudoGameScene -- LudoContext
    LudoGameScene --|> IContextManager_LudoContext_


    

    namespace LudoObjects{



        class Board{
            +List~Cell~ Cells : readonly
            +List~List~int~~ Paths : readonly

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
    Cell --* Board
    Cell -- CellType
    LudoContext *-- Board
    LudoContext *-- PlayerWithAction 

    Totem -- LudoTotemMoveTogether
    Totem -- SingleTotemActionable

    Totem -- Cell

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
    TotemRendering *-- Totem
    BoardRendering *-- Board
    LudoDiceRendering *-- LudoDice
    BoardRendering --|> ConsoleRenderable
    TotemRendering --|> ConsoleRenderable
    LudoDiceRendering --|> ConsoleRenderable







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
