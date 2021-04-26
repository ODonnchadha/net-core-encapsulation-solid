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
    public string WorkingDirectory { get; set; }
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

      A. Command: Mutate state.
        ```csharp
          public void Save(Order order);
          public void Send(T message);
          public void Associate(IFoo foo, Bar bar);
        ```
        - Somehow, we're changed the state of the system.
        - COMMON: They all return void(); Thus, the method *must* have a side-effect.
        ```csharp
          public string WorkingDirectory { get; set; }
          public void Save(int id, string message)
          {
            var path = GetFileName(id);
            File.WriteAllText(path, message);
          }
          // A proper query that has no side effects.
          public string GetFileName(int id)
          {
            return Path.Combine(this.WorkingDirectory, id + ".txt");
          }
        ```
        - NOTE: It is fine to query from a command, as the query has no side effects.

      B. Query: Do not mutate observable state.
        ```csharp
          public Order[] GetOrders(int usedId);
          public IFoo Map(Bar bar);
          public T Create();
        ```
        - COMMON: They all return something. They do not mutate state. 
        - Exhibit an example of idempotent behavior. 1:N times will not change the state of the system. 
        - "Asking the question should not change the answer."
        - This currently looks like a command but ought be a query.
        ```csharp
          public void Read(int id)
        ```
        - NOTE: An event is a side-effect.
          ```csharp
            public string Read(int id)
            {
              var path = Path.Combine(this.WorkingDirectory, id + ".txt");
              var msg = File.ReadAllText(path);
              // this.MessageRead(this, new MessageEventArgs { Message = msg });
              return msg;
            }
          ```

  - With this, we understand/reason about the high-level behavior of FileStore/code.
    - Trust that the code adheres to CQS.
    - How can you *trust* that a command accepts your input?
    - How can you *trust* a query to return a useful response?
  - Postel's Law:
    - Robustness principle: Be very conservative in what you send. Be very liberal in what you accept.
    - From a client's perspective: Guarentees are good. Stronger guarentees are better.
    - "If you can understand what a client meant, you should accept their input."
    - Fail fast. Inform the client right away and go to great lengths to inform the client where and what.
  - Currently, WorkingDirectory can be null.
  - It's very easy to create an instance of the object without the proper initialization occurring.
  ```csharp
    public string WorkingDirectory { get; set; }
  ```
  - Add a constructor. And a private set. And use a guard clause in the constructor.
  ```csharp
  public class FileStore
  {
    public FileStore(string workingDirectory)
    {
      if (workingDirectory == null) throw new ArgumentNullException("workingDirectory");
      this.WorkingDirectory = workingDirectory;
    }
    public string WorkingDirectory { get; private set; }
  }
  ```
  - Detour. Nullable versus Non-Nullable types.
  - Value types: Primitive. Non-nullable by default. With Generics, non-nullable.
  - Reference types: Nullable. This is a horrible, horrible invention. Tony Hoare is to blame.
  - Guard clauses allow us to fail fast. e.g. Check for valid path/directory.
  ```csharp
      if (!Directory.Exists(workingDirectory)) throw new ArgumentException("!Directory.Exists", "workingDirectory");
  ```
  - Output: Thinking about the post-conditions by way of ease and guarentee. Currenty, three (3) queries.
  - Try... catch is not great manner in which to implement control flow.
  - Method signature does not tell us whether null is valid/invalid. Null is really a bad design decesion as it removes trust.
  
  - This will lead for defensive coding. More lines of code for null check rather than for useful processing. 
  - Historical order: (And better.)
  - 1. Tester/Doer: If you have an operation and are unsure that the operation is legal:
    - An operation will come in pairs. e.g.: The Read() method will also come with a CanRead() method:
    ```csharp
      public bool CanRead(int id)
      {
        var path = this.GetFileName(id);
        return File.Exists(path);
      }
      public string Read(int id)
      {
        var path = this.GetFileName(id);
        if (!File.Exists(path)) throw;
        var message = File.ReadAllText(path);
        return message;
      }
    ```
    - NOTE: This is not threadsafe. In a multithreaded environment, you could get a 'true' back via CasnRead() and have another thread delete the file before your Read() method is invoked.
  - 2. TryRead idiom: (Atomic.)
    ```csharp
      public bool TryRead(int id, out message)
      {
        message = null;
        var path = this.GetFileName(id);
        if (!File.Exists(path)) throw;
        var message = File.ReadAllText(path);
        return true;
      }
    ```
    - This is not too OO. And this is not fluent at all. (Which is also not too OO.)
    - OOs say "Instead of having out parameters, we need to introduce some type of complex object." But the client will still need to view a property to see if something succeeded.
    - So we can still be null with the associated defensive coding.
  - 3. Maybe:
    - Borrowed concept from functional programming. Optional type. Collection with a specific constraint. Collection with 0-1 elements.
    - NOTE: IEnumerable<string> is not explicit enough. It's not a strong guarentee.
      ```csharp
        public class Maybe<T> : IEnumerable<T>
        {
          private readonly IEnumerable<T> values;
          public Maybe() => this.values = new T[0];
          public Maybe(T value) => this.values = new[] { value };
          public IEnumerator<t> GetEnumerator() => this.values.GetEnumerator();
          IEnumerator IEnumerable.GetEnumerator() => return this.GetEnumerator();
        }
        public Maybe<string> Read(int id, out message)
        {
          var path = this.GetFileName(id);
          if (!File.Exists(path)) throw;
          var message = File.ReadAllText(path);
          return new Maybe<string>(message);
        }
        var message = fileStore.Read(40).DefaultIfEmpty("").Single();
    ```
    - Guarentee you return the type expected. It may be empty, but it will not be null.

- SINGLE RESPONSIBILITY PRINCIPLE:
  - Supple code.
  - SOLID is not: Framework. Library. Pattern. 
  - It's not a goal. It's not technology-bound. Additionally, the order is not important.
  - The purpose of SOLID is to make one more production since the code is far more maintainable.
  - Design smells:
    1. Rigidity: Difficult to change.
    2. Fragility: Easy to break.
    3. Immobility: Difficult to reuse.
    4. Viscosity: Difficult to do the right thing.
    5. Needless complexity: Overdesign.

  ```csharp
    public class FileStore
    {
      public FileStore(DirectoryInfo workingDirectory);
      public DirectoryInfo WorkingDirectory { get; }
      public void Save(int id, string message);
      public Maybe<string> Read(int id);
      public FileInfo GetFileInfo(int id);
    }
  ```
  - SRP: A class should only have a single responsibility. Responsibility equals a reason to change.
  - "Each class should do one thing and do it well." e.g.: UNIX.
  - Consider the embedded logging, caching, and storage within the FileStore class. And orchestration.
  ```csharp
    public class MessageStore
    {
      private readonly StoreCache cache;
      private readonly StoreLogger logger;
      private readonly FileStore storage;

      public MessageStore(DirectoryInfo workingDirectory)
      {
        if (workingDirectory == null ) throw;
        if (!workingDirectory.Exists) throw;
        this.WorkingDirectory = workingDirectory;
        this.cache = new StoreCache();
        this.log = new StoreLogger();
        this.storage = new FileStore();
      }

      public DirectoryInfo WorkingDirectory { get; }
      public void Save(int id, string message);
      public Maybe<string> Read(int id);
      public FileInfo GetFileInfo(int id);
    }

    public class FileStore
    {
      public void WriteAllText(string path, string message);
      public string ReadAllText(string path);
      public FileInfo GetFileInfo(int id, string workingDirectory);
    }
  ```

- OPEN/CLOSED PRINCIPLE:
  - Needless complexity? Not with all five principles applied together.
  - "Developers have a tendency to attempt to solve specific problems with general solutions. This leads to coupling and complexity."
  - Are those skills, design skills and displine, available?
  - Instead of being general, code should be specific. Following the SRP, each concrete class is specific.
  - What if you need generality? Interfaces are discovered as we go along.
  - Reused abstractions principle: If you have abstractions, and they are not being reused, then you have poor abstractions.
  ```csharp
    public class FileStore: IStoreWriter, IStoreReader, IFileLocator {}
  ```
  - Abstraction is the elimination of the irrelevant and the amplification of the essential.
  - Do not design interfaces upfront. After a concrete class is developed, then look for common groupings and abstractions.
  - Interfaces are not designed; they are discovered.
  - Rule of three. Do not introduce a general purpose solution to something until you have three cases that fit into the general solution.
  - What about helper methods or classes? No. Rule of three. These are still general purpose solutions.

  - OCP: Open for extensibility and closed for modification. You are no longer allowed to make changes to that class. 
  - But anyone should be able to extend the class and modify its behavior. Bug fixing is still OK.
  - So this sounds extreme. However all clients will be affected. Every via a simple change.
  - Originally described with inheritance in mind. But this is not a great OO design as we should favor composition over inheritance.
  - e.g.: Favor composition over inheritance? Perhaps the strategy pattern is your friend? Or the compositie? Or the decorator?

  - Open for extensibility? e.g.: Like this:
  ```csharp
    public class FileStore
    {
      public virtual void WriteAllText(string path, string message);
      public virtual string ReadAllText(string path);
      public virtual FileInfo GetFileInfo(int id, string workingDirectory);
    }
  ```
  - (In Java, members are virtual by default. In Java and C#, classes themselves are not sealed by default.)
  - Factory method: Creates an instance of a polymophric class.

- LISKOV SUBSTITION PRINCIPLE:
  - Append-Only: CRUD: Create, Read, Update, Delete. These are the same operations as when you work with source code; however, Create & Read are apropos.
  - How do you evolve such a codebase like this? Perfect code all of the time?
  - Use a pattern called the Strangler pattern.
  - e.g.: X509Certificate class via Microsoft could not be moved forward. So, X509Certificate2 class. Also, the [Obsolete] attribute will emit a compiler warning.
  - LSP: Extensibility with inheritance. And now, polymorphism:
    - Subtypes must be substitutable for their base types. Or, consume any implementation without changing the correctness of the system.
  - When do we violate?
    - Most common: Throwing a "Not Supported" exception.
    - e.g.: .NET Framework. ICollection<T>. Add(), Clear(), Contains(), CopyTo(), Remove() interface.
    - ReadOnlyCollection<T>: ICollection<T>. Add(), Clear(), Remove().
    - Downcasts. Trying to check if ICollection<T> is actually List<T> or ReadOnlyCollection<T>. Not a guarentee, however.
    - Extracted interfaces. Most comman manner in which to violate. IDEs allow to extract an interface from a class. "Select all." Almost always problematic.
  - NOTE: LSP is often violated by attempts to remove features.
  - NOTE: Reused Abstractions Principle compliance indicates Liskov Substition Principle complaince. 
    - "You should have more than one implementation of a given interface." So, a given interface can be implemented by multiple concrete classes.
  ```csharp
    public void Save(int id, string message) 
    {
      this.Log.Saving(id);
      var file = this.GetFileInfo(id);
      this.Dtore.WriteAllText(file.FullName, message);
      this.Cache.AddOrUpdate(id, message);
      this.Log.Saved(id);
    }
    public Maybe<string> Read(int id)
    {
      thi.Log.Reading(id);
      var file = this.GetFileInfo(id);
      if (!file.Exists)
      {
        this.Log.DidNotFind(id);
        return new Maybe<string>();
      }
      var message = this.Cache.GtOrAdd(id, _ => this.Store.ReadAllText(file.FullName));
      this.Log.Returning(id);
      return new Maybe<string>(message);
    }
  ```
  - The store is defined by the FileStore class.
    ```csharp
    public class Filestore
    {
      public void WriteAllText(string path, string message) => File.WriteAllText(path, message);
      public string ReadAllText(string path) => File.ReadAllText(path);
      public FileInfo GetFileInfo(int id, string workingDirectory) => new FileInfo(Path.Combine(workingDirectory, id + ".txt"));
    }
  ```
  - Strange inheritance heirarchy, especially as we extract an interface.
  ```csharp
    public interface IStore
    {
      void WriteAllText(string path, string message);
      string ReadAllText(string path);
      FileInfo GetFileInfo(int id, string workingDirectory);
    }
    public class SqlStore: IStore
    {
      public override void WriteAllText(string path, string message);
      public override string ReadAllText(string path);
      // Throw new NotSupportedException.
      public override FileInfo GetFileInfo(int id, string workingDirectory) => base.GetFileInfo(id, workingDirectory);
    }
    public class MessageStore
    {
      private readonly IStore store;
      public MessageStore() => this.store = new FileStore();
    }
  ```
  - We need to remove the workingDirectory parameter from the interface.
  ```csharp
    public class Filestore : IStore
    {
      private readonly DirectoryInfo workingDirectory;
      public Filestore(DirectoryInfo workingDirectory) => this.workingDirectory = workingDirectory;
    }
    public interface IStore
    {
      void WriteAllText(int id, string message);
      Maybe<string> ReadAllText(int id);
      FileInfo GetFileInfo(int id);
    }
    public class SqlStore: IStore
    {
      public override void WriteAllText(string id, string message);
      public override string ReadAllText(int id);
      public override FileInfo GetFileInfo(int id, string workingDirectory) => Throw new NotSupportedException();
    }
  ```

- INTERFACE SEGRATION PRINCIPLE:
- DEPENDENCY INVERSION PRINCIPLE:
- COFFEE MAKER CODE EXAMPLE:
  - Uncle Bob's example.