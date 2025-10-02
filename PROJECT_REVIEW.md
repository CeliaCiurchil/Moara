# Moara Project - Comprehensive Review

## Executive Summary
**Moara** is a sophisticated implementation of the traditional Romanian board game "Nine Men's Morris" (Moara). This project demonstrates advanced software engineering practices through a well-architected C++ application featuring multiple user interfaces, AI opponents with different difficulty levels, and networked multiplayer capabilities.

---

## 1. Project Overview

### Game Description
Moara (Nine Men's Morris) is a strategy board game for two players. The game consists of three phases:
- **Placement Phase**: Players alternate placing pieces on the board
- **Movement Phase**: Players move pieces to adjacent positions
- **Capture Phase**: When a player forms a line (mill), they can capture an opponent's piece

### Project Statistics
- **Total Lines of Code**: ~17,700 lines
- **Languages**: C++ (primary), Protocol Buffers
- **Components**: 4 major modules (Core Logic, Frontend, Server, Client)
- **Test Coverage**: Comprehensive unit tests using Google Test framework

---

## 2. Core Skills & Technologies Demonstrated

### 2.1 Programming Languages & Frameworks
- **C++17/C++20**: Modern C++ with extensive use of:
  - Smart pointers (`std::shared_ptr`, `std::unique_ptr`)
  - Type aliases and templates
  - STL containers (`std::vector`, `std::list`, `std::pair`)
  - Lambda expressions and functional programming concepts
  
- **Qt Framework**: GUI development
  - Qt Widgets for desktop interface
  - Signal-slot mechanism for event handling
  - Custom painting and rendering
  - Resource management (`.qrc` files)

- **Protocol Buffers**: Serialization for network communication
  - Message definition (`.proto` files)
  - Client-server data exchange protocol

### 2.2 Software Design Patterns

#### **Strategy Pattern** ⭐
- **Location**: `IStrategy.h`, `EasyStrategy`, `MediumStrategy`, `CustomStrategy`
- **Purpose**: Implements different AI difficulty levels with interchangeable algorithms
- **Benefits**: Easy to add new AI strategies without modifying existing code

```cpp
class IStrategy {
    virtual std::pair<Pos, Pos> MovePiece(EPlayer player) = 0;
    virtual Pos CapturePiece(EPlayer player) = 0;
    virtual Pos PlacePiece(EPlayer player) = 0;
};
```

#### **Observer Pattern** ⭐
- **Location**: `IGameListener`, `IClientListener`
- **Purpose**: Decouples game logic from UI, enabling multiple observers
- **Implementation**: Listener lists with notification methods for game events

```cpp
class IGameListener {
    virtual void OnPlacePiece(Pos placePos, int color) = 0;
    virtual void OnRemovePiece(Pos removePos) = 0;
    virtual void OnMovePiece(Pos initialPos, Pos finalPos, int color) = 0;
    // ... more event handlers
};
```

#### **Singleton Pattern** ⭐
- **Location**: `Logger.h`
- **Purpose**: Single logging instance across the application
- **Implementation**: Thread-safe singleton with mutex protection

```cpp
class Logger {
public:
    static Logger& instance(const std::string& filename);
private:
    std::mutex logMutex;
};
```

#### **Factory Pattern** ⭐
- **Location**: `IGame::Produce()`, `IPiece` creation
- **Purpose**: Encapsulates object creation logic
- **Benefits**: Centralizes instantiation, enables testing modes

#### **Proxy Pattern** ⭐
- **Location**: `ClientProxy` in Server module
- **Purpose**: Manages client connections in client-server architecture
- **Benefits**: Abstracts network communication details

### 2.3 Object-Oriented Programming Principles

#### **Abstraction**
- Extensive use of interfaces (`IGame`, `IStrategy`, `IPiece`, `IGameListener`)
- Clear separation between interface and implementation
- Abstract base classes define contracts for derived classes

#### **Encapsulation**
- Private member variables with public getter/setter methods
- Data hiding in classes like `Board`, `Game`, `Piece`
- Protected access to internal state

#### **Inheritance**
- Multiple strategy implementations inherit from `IStrategy`
- `Piece` implements `IPiece` interface
- `Game` implements `IGame` interface

#### **Polymorphism**
- Virtual functions enable runtime polymorphism
- Strategy objects are used interchangeably via base pointer
- Listener callbacks demonstrate interface-based polymorphism

### 2.4 Advanced C++ Features

#### **Type Aliases**
```cpp
using Positions = std::vector<std::pair<int, int>>;
using Pos = std::pair<int, int>;
using Miliseconds = std::chrono::milliseconds;
using BoardMatrix = std::vector<std::vector<IPiecePtr>>;
```

#### **Smart Pointers**
- `std::shared_ptr` for shared ownership (pieces, strategies, game instances)
- Automatic memory management prevents memory leaks
- Clear ownership semantics

#### **Chrono Library**
- Time tracking for player moves
- 10-second turn timer implementation
- Precision timing using `std::chrono::milliseconds`

#### **STL Algorithms & Containers**
- Vector operations for board representation
- List for game history (undo functionality)
- Structured bindings: `auto [x, y] = position;`

### 2.5 Network Programming
- **Client-Server Architecture**: Multiplayer game support
- **Protocol Buffers**: Efficient binary serialization
- **SFML Networking**: Socket-based communication
- **Message Protocol**: Custom command system for game actions

### 2.6 Testing & Quality Assurance

#### **Google Test Framework**
- Comprehensive unit tests in `GameTests/`
- Mock objects using Google Mock (`MockGameListener`)
- Test fixtures for setup/teardown
- Over 20+ test cases covering:
  - Piece placement and movement
  - Capture mechanics
  - Undo functionality
  - Save/Load operations
  - Win conditions

```cpp
TEST_F(GameTest, Place1PieceNotifiesListeners) {
    EXPECT_CALL(listener, OnPlacePiece(Pos{0, 0}, 0)).Times(1);
    game.PlacePiece({0, 0});
}
```

---

## 3. Architecture & System Design

### 3.1 Modular Architecture

```
Moara/
├── Moara/           # Core game logic (business layer)
├── FrontEnd/        # Qt-based desktop GUI
├── Server/          # Multiplayer game server
├── Client/          # Network client module
├── GameTests/       # Unit and integration tests
└── MainApp/         # Console application entry point
```

### 3.2 Core Components

#### **Game Engine (`Game.h`, `Game.cpp`)**
- Central game controller implementing `IGame` interface
- State machine managing game phases (Place, Move, Capture, Win)
- Turn-based player management
- Timer integration for move timeouts
- History tracking for undo functionality

#### **Board Management (`Board.h`, `Board.cpp`)**
- 3x8 matrix representation (3 concentric squares, 8 positions each)
- Piece tracking and validation
- Line detection algorithm
- Move validation and blockage detection
- Statistics tracking (pieces per player, lines formed)

#### **Piece System (`IPiece.h`, `Piece.h`)**
- Position management
- Color identification
- Movement validation based on board structure
- Abstract interface for extensibility

#### **AI System (Strategy Pattern)**
- **Easy Strategy**: Random valid moves
- **Medium Strategy**: Strategic move selection (block opponent lines, form own lines)
- **Custom Strategy**: Template for user-defined AI

#### **Logger (`Logger.h`)**
- Thread-safe file logging
- Timestamped entries
- Multiple message types (Info, Error, Warning, Debug)
- Singleton pattern ensures single log file

### 3.3 Data Flow

```
User Input → UI Layer (Qt/Console) → Game Controller → Board Logic → Piece Validation
                                            ↓
                                      Listener Notifications
                                            ↓
                                    UI Update (Observer Pattern)
```

### 3.4 State Management

#### **Game States** (`EState.h`)
- `Place`: Initial piece placement phase
- `Move`: Piece movement phase
- `Capture`: Opponent piece capture phase
- `Player1Won` / `Player2Won`: Terminal states

#### **State Transitions**
```
Place → (Line formed) → Capture → Place/Move
Move → (Line formed) → Capture → Move
Move → (No valid moves) → PlayerXWon
```

---

## 4. Key Features & Functionality

### 4.1 Game Modes
1. **Player vs Player**: Two human players
2. **Player vs Computer**: Single player with AI opponent
3. **Online Multiplayer**: Network-based play via client-server

### 4.2 AI Difficulty Levels
- **Easy**: Random move selection from valid options
- **Medium**: Strategic decision making (offensive and defensive play)
- **Custom**: Extensible strategy interface for advanced implementations

### 4.3 Game Management
- **Save/Load**: Persist game state to files
- **Undo**: Step-by-step action reversal with history tracking
- **Reset**: New game initialization
- **Timer**: 10-second move limit with automatic loss on timeout

### 4.4 User Interface Features
- **Visual Board**: Qt-based graphical representation
- **Drag & Drop**: Intuitive piece movement
- **Hints System**: Visual indicators for valid moves
- **Status Display**: Current player, game state, timer
- **Strategy Selection**: Dropdown to choose AI difficulty

### 4.5 Networking Features
- **Session Management**: Multiple concurrent games
- **Real-time Updates**: Synchronous board state across clients
- **Reconnection Handling**: Graceful disconnect management
- **Protocol Buffers**: Efficient binary message protocol

---

## 5. Code Quality & Best Practices

### 5.1 Documentation
- **Comprehensive XML Comments**: All public APIs documented
- **Summary Tags**: Clear descriptions of methods and parameters
- **Remarks Sections**: Additional context and usage examples
- **Return Value Documentation**: Expected outcomes clearly stated

### 5.2 Error Handling
- **Enum-based Error Codes** (`EOperationResult`): 
  - `NoError`, `NoPiece`, `WrongPlayer`, `InvalidPosition`, etc.
- **Defensive Programming**: Input validation before processing
- **Exception Safety**: RAII principles with smart pointers

### 5.3 Code Organization
- **Header Guards**: `#pragma once` consistently used
- **Forward Declarations**: Minimize compilation dependencies
- **Namespace Management**: Implicit use of project namespaces
- **File Structure**: Clear separation of interface (`.h`) and implementation (`.cpp`)

### 5.4 Naming Conventions
- **Classes**: PascalCase (`BoardWidget`, `CustomStrategy`)
- **Methods**: PascalCase (`PlacePiece`, `GetCurrentPlayer`)
- **Member Variables**: `m_` prefix with camelCase (`m_board`, `m_currentPlayer`)
- **Enums**: `E` prefix (`EPlayer`, `EState`, `EColor`)
- **Interfaces**: `I` prefix (`IGame`, `IStrategy`, `IPiece`)

### 5.5 Memory Management
- **Smart Pointers**: No manual `new`/`delete` in modern code
- **RAII**: Resource acquisition is initialization pattern
- **No Memory Leaks**: Verified through testing

---

## 6. Testing Strategy

### 6.1 Test Coverage
- **Unit Tests**: Individual component testing (Board, Piece, Game)
- **Integration Tests**: Multi-component interaction testing
- **Mock Objects**: Isolate components during testing
- **State Verification**: Ensure correct game state transitions

### 6.2 Test Organization
```cpp
class GameTest : public ::testing::Test {
protected:
    void SetUp() override {
        // Test initialization
    }
};

TEST_F(GameTest, Place1PieceNotifiesListeners) {
    // Arrange, Act, Assert pattern
}
```

### 6.3 Test Files
- `GameTests.cpp`: Core game logic tests
- `BoardTests.cpp`: Board manipulation tests
- `PieceTests.cpp`: Piece behavior tests
- Test data files: Predefined game states for loading

---

## 7. Technologies & Tools

### 7.1 Development Tools
- **Visual Studio**: Primary IDE (`.vcxproj` files)
- **MSBuild**: Build system
- **Git**: Version control
- **NuGet**: Package management (Google Test)

### 7.2 Libraries & Frameworks
- **Qt 6**: Cross-platform GUI framework
- **Google Test**: Unit testing framework
- **Google Mock**: Mocking framework
- **SFML Network**: Socket-based networking
- **Protocol Buffers**: Serialization library
- **STL**: C++ Standard Template Library

### 7.3 Build System
- **Visual Studio Solution** (`.sln`): Multi-project configuration
- **Project Files** (`.vcxproj`): Individual component configuration
- **Resource Files** (`.qrc`): UI assets and resources

---

## 8. Network Protocol Design

### 8.1 Protocol Architecture
- **Binary Protocol**: Protocol Buffers for efficiency
- **Command-Based**: Discrete action messages
- **Synchronous Updates**: Real-time state synchronization

### 8.2 Message Types
```protobuf
enum ECommand {
    PLACE_PIECE,
    MOVE_PIECE,
    CAPTURE_PIECE,
    UNDO,
    RESET,
    GET_STATE,
    SET_STRATEGY,
    // ... more commands
}
```

### 8.3 Client-Server Communication
```
Client → [Message] → Server → [Process] → [Broadcast] → All Clients
```

---

## 9. Notable Implementation Details

### 9.1 Board Representation
- **Matrix Structure**: 3 rows × 8 columns representing concentric squares
- **Position Encoding**: `(row, column)` pairs
- **Adjacency Rules**: Piece can move to valid adjacent positions

### 9.2 Line Detection Algorithm
```cpp
bool CheckLine(Pos position, int color) const {
    // Checks horizontal, vertical lines
    // Returns true if a mill (3 in a row) is formed
}
```

### 9.3 Undo Mechanism
- **History Stack**: `std::list<GameHistory>`
- **Action Recording**: State and positions saved for each move
- **Reverse Operations**: Undo place, move, capture actions

### 9.4 Timer System
- **Countdown Timer**: 10 seconds per move
- **Automatic Loss**: Timeout triggers win condition for opponent
- **Display Integration**: Real-time timer display in UI

---

## 10. Extensibility & Future Enhancements

### 10.1 Extensible Components
- **Strategy Interface**: Easy to add new AI algorithms
- **Listener Pattern**: Simple to add new UI representations
- **Network Protocol**: Versioned for backward compatibility

### 10.2 Potential Enhancements
- Tournament mode with multiple rounds
- Advanced AI using minimax or Monte Carlo tree search
- Replay system for reviewing games
- Ranking and matchmaking system
- Mobile client support
- Web-based interface

---

## 11. Skills Summary

This project demonstrates proficiency in:

### **Software Engineering**
✓ Design Patterns (Strategy, Observer, Singleton, Factory, Proxy)  
✓ SOLID Principles (Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion)  
✓ Object-Oriented Design  
✓ Modular Architecture  
✓ Code Documentation  

### **C++ Development**
✓ Modern C++ (C++17/20 features)  
✓ Smart Pointers & Memory Management  
✓ STL Containers & Algorithms  
✓ Templates & Type Aliases  
✓ Lambda Expressions  
✓ RAII Pattern  

### **GUI Development**
✓ Qt Framework  
✓ Event-Driven Programming  
✓ Signal-Slot Mechanism  
✓ Custom Widgets  
✓ Resource Management  

### **Network Programming**
✓ Client-Server Architecture  
✓ Socket Programming (SFML)  
✓ Protocol Design  
✓ Binary Serialization (Protocol Buffers)  
✓ Message-Based Communication  

### **Testing**
✓ Unit Testing (Google Test)  
✓ Mocking (Google Mock)  
✓ Test-Driven Development  
✓ Test Fixtures  
✓ Integration Testing  

### **Game Development**
✓ Game State Management  
✓ Turn-Based Game Logic  
✓ AI Implementation  
✓ Save/Load Systems  
✓ Undo/Redo Functionality  

### **Tools & Practices**
✓ Visual Studio IDE  
✓ Git Version Control  
✓ Build Systems (MSBuild)  
✓ Debugging Techniques  
✓ Code Review Practices  

---

## 12. Conclusion

The **Moara** project is an exemplary demonstration of professional software engineering practices applied to game development. It showcases:

1. **Clean Architecture**: Well-organized, modular design with clear separation of concerns
2. **Advanced C++ Skills**: Modern C++ features, smart memory management, and STL proficiency
3. **Design Pattern Mastery**: Practical application of multiple design patterns
4. **Cross-Platform Development**: Qt framework for portable GUI applications
5. **Network Programming**: Complete client-server multiplayer implementation
6. **Testing Excellence**: Comprehensive test coverage with industry-standard tools
7. **Code Quality**: Well-documented, maintainable, and extensible codebase

This project effectively demonstrates the technical competence required for senior-level software engineering positions, particularly in game development, systems programming, or application development roles.

---

**Project Repository**: [CeliaCiurchil/Moara](https://github.com/CeliaCiurchil/Moara)  
**Total LOC**: ~17,700 lines  
**Primary Language**: C++ with Qt Framework  
**Test Framework**: Google Test & Google Mock  
**Network Protocol**: Protocol Buffers over SFML Sockets
