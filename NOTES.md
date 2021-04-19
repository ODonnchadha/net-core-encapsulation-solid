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


- SINGLE RESPONSIBILITY PRINCIPLE:
- OPEN/CLOSED PRINCIPLE:
- LISKOV SUBSYITION PRINCIPLE:
- INTERFACE SEGRATION PRINCIPLE:
- DEPENDENCY INVERSION PRINCIPLE:

- COFFEE MAKER CODE EXAMPLE:
  - Uncle Bob's example.