## Culled from a Pluralsight class entitled "Encapsulation & SOLID."
## By Mark Seemann

- INTRODUCTION:
  - Write better code: Maintainable. Readable. Something that humans can reason about.

- ENCAPSULATION:
  - Fundamental.
  - Do you build applications on top of reuseable components? e.g.: Caching. DI Container.
  - And do you look at the associated source code of those reuseable components?
  - Write code for stupid programmers.
  ```csharp
    public class FileStore
    {
      public string WorkingDirectory { get; set; }
      public string Save(int id, string message);
      public event EventHandler<MessageEventArgs> MessageRead;
      public void Read(int id);
    }
  ```
  - Why string on Save()? Why void on Read()? We must "read" the source code:
  ```csharp
    public string Save(int id, string message)
    {
      var path = Path.Combine(this.WorkingDirectory, id + ".txt");
      File.WriteAllText(path, message);
      return path;
    }
    public void Read(int id)
    {
      var path = Path.Combine(this.WorkingDirectory, id + ".txt");
      var msg = File.ReadAllText(path);
      this.MessageRead(this, new MessageEventArgs { Message = msg });
    }
  ```
  - Just tell your manager that the code sucks. End of story.
  - Worry about long-term productivity later, maintainability, and technical debt later.
  - NOTE: Code is read more than written. 10x. 20x.
  - Classic definition: Two (2) traits:
    1. Information hiding: This has nothing to do with exposing properties vis getters and setters.
      - New phrase: "Implementation hiding." We hide *some* of the information.
    2. Protection Of Invariants: Checking for preconditions and guarenteeing for post-conditions.
      - Invalid states are impossible. Write classes so that it is difficult to place classes in invalid states.
      - Pre- and post-conditions are also know as assertions.
      - Programatically adding checks to the code is known as protection of invariants.
  - Beyond object-oriented design for some concrete and actionable principles:
    1. Command Query Seperation: Operations should either be commands or queries but not both. 
      - NOT CQRS. This is an architectural distribution design pattern that shares some language with CQR.
      - Commands have an observable side effect in the system.
      - Operations that mutate the observable state.
      - Query is an operation that returns data.


- SINGLE RESPONSIBILITY PRINCIPLE:
- OPEN/CLOSED PRINCIPLE:
- LISKOV SUBSYITION PRINCIPLE:
- INTERFACE SEGRATION PRINCIPLE:
- DEPENDENCY INVERSION PRINCIPLE:

- COFFEE MAKER CODE EXAMPLE:
  - Uncle Bob's example.