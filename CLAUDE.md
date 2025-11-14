# CLAUDE.md - AI Assistant Guide

## Project Overview

This repository contains the **O'Reilly Reactive Architecture Course** materials, which teaches reactive programming concepts using the Akka actor framework with Java. The project is structured as a series of progressive exercises that build a simulated "Coffee House" application to demonstrate actor-based concurrent programming.

**Repository**: oreilly-reactive-with-akka
**Language**: Java (with Scala build tooling)
**Framework**: Akka 2.5.3
**Build System**: SBT 0.13.15
**Java Version**: JVM 1.8+
**Scala Version**: 2.12.2

---

## Repository Structure

### Root Directory Layout

```
oreilly-reactive-with-akka/
├── README.md                          # Setup and course instructions
├── build.sbt                          # Main SBT build configuration
├── project/                           # SBT project configuration
│   ├── build.properties               # SBT version (0.13.15)
│   └── plugins.sbt                    # SBT plugins (Eclipse)
├── man.sbt                           # Custom 'man' command for viewing instructions
├── navigation.sbt                     # Custom 'next'/'prev' navigation commands
├── shell-prompt.sbt                   # Custom shell prompt configuration
├── exercise_000_Initial_state/        # Exercise 0: Starting point
├── exercise_001_Implement_actor/      # Exercise 1: Basic actor implementation
├── exercise_002_Top_level_actor/      # Exercise 2: Top-level actor creation
├── exercise_003_Message_actor/        # Exercise 3: Actor messaging
├── exercise_004_Use_sender/           # Exercise 4: Using sender()
├── exercise_005_Child_actors/         # Exercise 5: Child actor creation
├── exercise_006_Actor_state/          # Exercise 6: Managing actor state
├── exercise_007_Use_scheduler/        # Exercise 7: Akka scheduler usage
├── exercise_008_Busy_actor/           # Exercise 8: Handling busy actors
├── exercise_009_Stop_actor/           # Exercise 9: Actor lifecycle
├── exercise_010_Lifecycle_monitoring/ # Exercise 10: Death watch
├── exercise_011_Faulty_guest/         # Exercise 11: Fault tolerance
├── exercise_012_Custom_supervision/   # Exercise 12: Supervision strategies
├── exercise_013_Faulty_waiter/        # Exercise 13: Advanced supervision
├── exercise_014_Self_healing/         # Exercise 14: Self-healing systems
├── exercise_015_Detect_bottleneck/    # Exercise 15: Performance monitoring
└── exercise_016_Use_router/           # Exercise 16: Router patterns
```

### Exercise Directory Structure

Each exercise follows this standard structure:

```
exercise_XXX_Name/
├── README.md                          # Exercise-specific instructions
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/lightbend/training/coffeehouse/
│   │   │       ├── CoffeeHouseApp.java     # Main application entry point
│   │   │       ├── CoffeeHouse.java        # Top-level actor
│   │   │       ├── Guest.java              # Guest actor
│   │   │       ├── Waiter.java             # Waiter actor
│   │   │       ├── Barista.java            # Barista actor
│   │   │       ├── Coffee.java             # Coffee message class
│   │   │       ├── Terminal.java           # Terminal interface
│   │   │       └── TerminalCommand.java    # Command parsing
│   │   └── resources/
│   │       ├── application.conf            # Akka configuration
│   │       └── logback.xml                 # Logging configuration
│   └── test/
│       ├── java/
│       │   └── com/lightbend/training/coffeehouse/
│       │       ├── BaseAkkaTestCase.java   # Test base class
│       │       └── *Test.java              # JUnit test classes
│       └── resources/
│           ├── README.md                   # Exercise instructions (detailed)
│           └── logback-test.xml            # Test logging config
```

---

## Key Architectural Patterns

### Actor Model

The Coffee House application demonstrates the actor model with these core actors:

1. **CoffeeHouse** (Top-level actor)
   - Manages the entire coffee house system
   - Creates and supervises child actors (Waiter, Barista, Guests)
   - Implements supervision strategies for fault tolerance
   - Tracks caffeine consumption per guest

2. **Guest** (Child actor)
   - Represents a customer ordering coffee
   - Sends coffee orders to the Waiter
   - Consumes coffee and becomes "caffeinated"
   - Can throw CaffeineException when over-caffeinated

3. **Waiter** (Child actor)
   - Takes orders from Guests
   - Forwards orders to Barista
   - Delivers coffee back to Guests
   - Can become "frustrated" and throw exceptions

4. **Barista** (Child actor)
   - Prepares coffee orders
   - Uses configurable preparation time
   - Can make mistakes (configurable accuracy)
   - Often uses router patterns for scaling (Exercise 16)

### Message Flow

```
Guest → Waiter.ServeCoffee → CoffeeHouse.ApproveCoffee → Barista.PrepareCoffee → Waiter → Guest.CoffeeServed
```

### Supervision Hierarchy

```
CoffeeHouse (OneForOneStrategy)
├── Waiter (custom strategy for FrustratedException)
├── Barista (often a router pool in later exercises)
└── Guest(s) (stopped on CaffeineException)
```

---

## Build System Details

### SBT Configuration (build.sbt)

**Key Dependencies:**
- `akka-actor` 2.5.3 - Core actor framework
- `akka-remote` 2.5.3 - Remote actors support
- `akka-slf4j` 2.5.3 - Logging integration
- `akka-testkit` 2.5.3 - Testing utilities
- `logback-classic` 1.2.3 - Logging implementation
- `guava` 22.0 - Google common utilities
- `junit-interface` 0.11 - JUnit integration
- `assertj-core` 3.8.0 - Fluent assertions

**Build Settings:**
- Java source only (no Scala sources)
- JVM target: 1.8
- Parallel test execution: disabled
- Test buffering: disabled
- Eclipse integration via sbteclipse plugin

### Custom SBT Commands

1. **`man`** - Display setup instructions
2. **`man e`** - Display current exercise instructions
3. **`next`** - Navigate to next exercise
4. **`prev`** - Navigate to previous exercise
5. **`run`** - Start the CoffeeHouseApp for current exercise
6. **`test`** - Run automated tests for current exercise

---

## Coding Conventions

### Java Style

1. **Package Structure**: All classes in `com.lightbend.training.coffeehouse`
2. **Actor Implementation**: Extend `AbstractLoggingActor` for built-in logging
3. **Props Pattern**: Always use static `props()` methods for actor creation
4. **Message Classes**: Immutable final classes with proper equals/hashCode
5. **Preconditions**: Use Guava's `checkNotNull()` for validation
6. **Copyright Headers**: Typesafe copyright on all source files

### Actor Patterns

```java
// Standard actor structure
public class MyActor extends AbstractLoggingActor {

    // Static factory method for Props
    public static Props props(/* parameters */) {
        return Props.create(MyActor.class, () -> new MyActor(/* args */));
    }

    // Constructor
    public MyActor(/* parameters */) {
        // Initialize actor state
    }

    // Receive behavior
    @Override
    public Receive createReceive() {
        return receiveBuilder()
            .match(MessageClass.class, this::handleMessage)
            .matchAny(this::unhandled)
            .build();
    }

    // Message handler
    private void handleMessage(MessageClass msg) {
        // Handle message
    }
}
```

### Message Class Pattern

```java
public static final class MyMessage {
    public final String field1;
    public final int field2;

    public MyMessage(final String field1, final int field2) {
        checkNotNull(field1, "field1 cannot be null");
        this.field1 = field1;
        this.field2 = field2;
    }

    @Override
    public boolean equals(Object o) { /* implementation */ }

    @Override
    public int hashCode() { /* implementation */ }

    @Override
    public String toString() { /* implementation */ }
}
```

### Configuration Pattern

Configuration is loaded from `application.conf` using Typesafe Config:

```java
private final FiniteDuration duration = Duration.create(
    context().system().settings().config()
        .getDuration("coffee-house.some.duration", MILLISECONDS),
    MILLISECONDS);

private final int someValue =
    context().system().settings().config()
        .getInt("coffee-house.some.value");
```

---

## Testing Conventions

### Test Structure

1. **Base Class**: All tests extend `BaseAkkaTestCase`
2. **Test Framework**: JUnit with Akka TestKit
3. **Assertions**: AssertJ for fluent assertions
4. **Event Filtering**: Mute log messages during tests

### Common Test Patterns

```java
public class MyActorTest extends BaseAkkaTestCase {

    @Test
    public void testSomeBehavior() {
        new JavaTestKit(system) {{
            // Create actor
            final ActorRef actor = system.actorOf(MyActor.props());

            // Send message
            actor.tell(new MyMessage(), getRef());

            // Expect response
            expectMsgEquals(duration("1 second"), expectedResponse);
        }};
    }
}
```

### Intercepting Log Messages

```java
interceptInfoLogMessage(kit, "expected log pattern", 1, () -> {
    // Code that should log the message
    actor.tell(message, kit.getRef());
});
```

### Verifying Actor Creation

```java
ActorRef actor = expectActor(kit, "user/coffee-house/guest");
```

---

## Configuration Files

### application.conf

Located in `src/main/resources/application.conf`:

```hocon
akka {
  loggers = [akka.event.slf4j.Slf4jLogger]
  loglevel = DEBUG

  actor {
    debug {
      lifecycle = on
      unhandled = on
    }
    deployment {
      # Router configurations (Exercise 16)
      /coffee-house/barista {
        router = round-robin-pool
        nr-of-instances = 4
      }
    }
  }
}

coffee-house {
  caffeine-limit = 1000
  barista {
    prepare-coffee-duration = 2 seconds
    accuracy = 100
  }
  waiter {
    max-complaint-count = 2
  }
  guest {
    finish-coffee-duration = 2 seconds
  }
}
```

### logback.xml

Logging configuration using Logback with console appender and pattern layout.

---

## Development Workflow

### Working with Exercises

1. **Starting SBT**: Run `sbt` from the project root
2. **View Instructions**: Use `man` to see setup, `man e` for current exercise
3. **Navigate**: Use `next` to move forward, `prev` to go back
4. **Implement**: Edit Java files in `src/main/java/`
5. **Test**: Run `test` to verify solution
6. **Run**: Use `run` to interact with the application
7. **Compile**: Use `compile` to check for compilation errors

### Interactive Application

The CoffeeHouseApp provides a terminal interface:

- **`guest N [coffee] [limit]`** - Create N guests with optional coffee preference and caffeine limit
- **`status`** - Show system status
- **`q` or `quit`** - Shutdown the application

### Exercise Progression

Each exercise builds on the previous one:

0. **Initial State** - Project skeleton
1. **Implement Actor** - Create first actor
2. **Top Level Actor** - Create CoffeeHouse
3. **Message Actor** - Implement messaging
4. **Use Sender** - Use sender() for replies
5. **Child Actors** - Create child actors
6. **Actor State** - Manage mutable state
7. **Use Scheduler** - Schedule periodic tasks
8. **Busy Actor** - Handle mailbox pressure
9. **Stop Actor** - Manage lifecycle
10. **Lifecycle Monitoring** - Death watch
11. **Faulty Guest** - Handle exceptions
12. **Custom Supervision** - Supervision strategies
13. **Faulty Waiter** - Advanced fault handling
14. **Self Healing** - Automatic recovery
15. **Detect Bottleneck** - Performance monitoring
16. **Use Router** - Scaling with routers

---

## Common Pitfalls and Best Practices

### Actor Best Practices

1. **Immutability**: Message classes should be immutable
2. **Props Factory**: Always use `props()` static methods
3. **No Blocking**: Never block inside an actor's receive method
4. **Supervision**: Design supervision strategies carefully
5. **Testing**: Use TestProbe for actor interactions in tests
6. **Lifecycle**: Clean up resources in `postStop()`
7. **Tell vs Ask**: Prefer `tell()` over `ask()` for better performance

### Configuration Best Practices

1. **Externalize**: Keep configuration in `application.conf`
2. **Defaults**: Provide sensible defaults
3. **Duration**: Use Scala Duration with TimeUnit
4. **Deployment**: Router configurations go in `akka.actor.deployment`

### Testing Best Practices

1. **Isolation**: Each test should be independent
2. **Muting**: Mute expected log messages
3. **Timeouts**: Use appropriate timeouts for expectations
4. **TestKit**: Leverage JavaTestKit utilities
5. **Event Filtering**: Verify log messages when needed

---

## AI Assistant Guidelines

### When Modifying Code

1. **Preserve Structure**: Keep the exercise structure intact
2. **Follow Patterns**: Use established actor and message patterns
3. **Check Tests**: Always run tests after changes
4. **Incremental**: Make changes incrementally per exercise
5. **Configuration**: Update `application.conf` when needed
6. **Logging**: Maintain debug logging for visibility

### When Analyzing Code

1. **Exercise Context**: Consider which exercise you're in
2. **Progressive Learning**: Earlier exercises have incomplete implementations
3. **Comments**: Look for `@todo` and `ANSWER` markers in code
4. **Tests**: Tests often reveal expected behavior
5. **README**: Check exercise README for requirements

### When Debugging

1. **Logs**: Check logback.xml configuration
2. **Test Output**: Review test failures carefully
3. **Actor Hierarchy**: Understand parent-child relationships
4. **Message Flow**: Trace message paths through actors
5. **Supervision**: Check supervision strategies for fault handling

### When Adding Features

1. **Actor Messages**: Define immutable message classes
2. **Props Pattern**: Use factory methods
3. **Configuration**: Add settings to application.conf
4. **Tests**: Write tests before implementation
5. **Documentation**: Update exercise README if needed

### Commands to Avoid

- Do NOT use `reload` unless you've modified build files
- Do NOT use parallel execution (it's disabled)
- Do NOT modify SBT build files unless specifically requested
- Do NOT change the multi-module structure

### Git Workflow

This is a learning repository, so:
- Commits should reflect exercise completion
- Each exercise can be a logical commit point
- Branch names should indicate the exercise being worked on
- Keep commit messages descriptive of the learning goal

---

## Quick Reference

### File Locations

- **Main Entry Point**: `src/main/java/com/lightbend/training/coffeehouse/CoffeeHouseApp.java`
- **Top Actor**: `src/main/java/com/lightbend/training/coffeehouse/CoffeeHouse.java`
- **Config**: `src/main/resources/application.conf`
- **Tests**: `src/test/java/com/lightbend/training/coffeehouse/*Test.java`

### Key Classes

- `CoffeeHouseApp` - Application entry point with Terminal
- `CoffeeHouse` - Top-level supervisor actor
- `Guest` - Customer actor
- `Waiter` - Order-taking actor
- `Barista` - Coffee-making actor
- `Coffee` - Immutable coffee type enum
- `BaseAkkaTestCase` - Test base class

### Important Concepts

- **Actor System**: ActorSystem.create()
- **Actor Creation**: context().actorOf()
- **Messaging**: actorRef.tell(message, sender())
- **Death Watch**: context().watch(actorRef)
- **Supervision**: supervisorStrategy() override
- **Scheduling**: context().system().scheduler()
- **Routing**: FromConfig.getInstance().props()

---

## Resources

- **Akka Documentation**: https://doc.akka.io/docs/akka/2.5/
- **Reactive Architecture**: https://www.reactivemanifesto.org/
- **SBT Documentation**: https://www.scala-sbt.org/0.13/docs/
- **Lightbend Training**: https://www.lightbend.com/learn

---

## Version Information

- **Last Updated**: 2025-11-14
- **Akka Version**: 2.5.3
- **SBT Version**: 0.13.15
- **Java Version**: 1.8+
- **Scala Version**: 2.12.2

---

This CLAUDE.md file should be updated as the codebase evolves or new patterns emerge.
