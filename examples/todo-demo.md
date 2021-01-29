## TODO - CRUD demo

```
OutOfRange = RuntimeException;
```
```
TodoItemId = Int => {
	if (value <= 0) {
		throw OutOfRange();
	}
};
```
```
TodoItem = {
	id: TodoItemId;
	content: String;
	isDone: Bool;
};
```
```
TodoItemRepository = !{
	persist    : (item: TodoItem)     => ();
	remove     : (itemId: TodoItemId) => ();
	ofId       : (itemId: TodoItemId) => TodoItem?;
	all        : (limit: Int?)        => TodoItem*;
	nextItemId : ()                   => TodoItemId;
};		
```
```
TodoItemCommandFactory = !{
	todoUseCase  : () => TodoUseCase;
	doUseCase    : () => DoUseCase;
	undoUseCase  : () => UndoUseCase;
	renameUseCase: () => RenameUseCase;
	removeUseCase: () => RemoveUseCase;
};
```
```
TodoItemQueryFactory = !{
	listUseCase  : () => ListUseCase;
	viewUseCase  : () => ViewUseCase;
};
```
```
TodoCommand      = {                     content: String; };
DoCommand        = { todoItemId: String; };
UndoCommand      = { todoItemId: String; };
UpdateCommand    = { todoItemId: String; content: String; };
RemoveCommand    = { todoItemId: String; };
```
```
ListQuery        = { itemsCount: Int?;   };
ViewQuery        = { todoItemId: String; };
```
```
TodoUseCase      = !{ execute: (command: TodoCommand)   => String; }
DoUseCase        = !{ execute: (command: DoCommand)     => (); }
UndoUseCase      = !{ execute: (command: UndoCommand)   => (); }
UpdateUseCase    = !{ execute: (command: UpdateCommand) => (); }
RemoveUseCase    = !{ execute: (command: RemoveCommand) => (); }
```
```
ListUseCase      = !{ execute: (query  : ListQuery)     => TodoItem*; }
ViewUseCase      = !{ execute: (query  : ViewQuery)     => TodoItem?; }
```
```
TodoHandler      = {
	implements TodoUseCase;
	repository: TodoItemRepository!;
	execute = repository.save(TodoItem(
		id: repository.nextItemId(),
		content: command.content,
		isDone: false
	));
};
DoHandler        = {
	implements DoUseCase;
	repository: TodoItemRepository!;
	execute = repository.ofId(command.todoItemId) is {
		item: TodoItem => repository.save(item.markAsDone()),
		None => ()
	};
};
UpdateHandler        = {
	implements UpdateUseCase;
	repository: TodoItemRepository!;
	execute = repository.ofId(command.todoItemId) is {
		item: TodoItem => repository.save(item.updateContentTo(command.content)),
		None => ()
	};
};
RemoveHandler        = {
	implements RemoveUseCase;
	repository: TodoItemRepository!;
	execute = repository.ofId(command.todoItemId) is {
		item: TodoItem => repository.remove(item.id),
		None => ()
	};
};
```
```
TodoItemNotFound = Exception;
```
```
TodoItemFactory = {
	implements TodoItemCommandFactory;
	implements TodoItemQueryFactory;
	
	todoUseCase  : () => TodoUseCase = TodoHandler();
	doUseCase    : () => DoUseCase = DoHandler();
	undoUseCase  : () => UndoUseCase = UndoHandler();
	renameUseCase: () => RenameUseCase = RenameHandler();
	removeUseCase: () => RemoveUseCase = RemoveHandler();
	
	listUseCase  : () => ListUseCase = ListHandler();
	viewUseCase  : () => ViewUseCase = ViewHandler();
};
```
```
Serializer = !{
	serialize: (value: Any) => String;
	unserialize: (value: String) => Any;
};
```
```
Redis = !{
	exists: (key: String) => Bool;
	get: (key: String) => String?;
	set: (key: String, value: String) => ();
	remove: (key: String) => ();
};
```
```
RedisTodoItemRepository = {
	implements TodoItemRepository;
	
	redis: Redis!;
	serializer: Serializer!;
	uuid: UUID!;
	
	ITEMS_KEY: String = "items";

	retrieve : () => TodoItem# {
		value: Any = redis.exists(ITEMS_KEY) ? 
			serializer.unserialize(redis.get(ITEMS_KEY)) : None;
		return value is {
			TodoItem# => value,
			Any => TodoItem#{}
		};
	};
	store: (data: TodoItem#) => () {		
		redis.set(ITEMS_KEY, serializer.serialize(data));
	};
	
	persist    : (item: TodoItem)     => () = store(retrieve().put(item.id, item));
	remove     : (itemId: TodoItemId) => () = store(retrieve().remove(item.id));
	ofId       : (itemId: TodoItemId) => TodoItem? = retrieve().get(item.id);
	all        : (limit: Int?)        => TodoItem* = retrieve().values();
	nextItemId : ()                   => TodoItemId = TodoItemId(uuid.generate());
};
```
```
InMemoryTodoItemRepository = {
	implements TodoItemRepository;
	
	~items: TodoItem#;
	
	persist    : (item: TodoItem)     => () = {
		~items = ~items.put(item.id, Item);
	};
	remove     : (itemId: TodoItemId) => () = {
		item: TodoItem? = ~items[String(itemId)];
		item is {
			t: TodoItem => ~items = ~items.remove(String(itemId));
			None => throw TodoItemNotFound()
		}
	};
	ofId       : (itemId: TodoItemId) => TodoItem? = ~items[String(itemId)];		
	all        : (limit: Int?)        => TodoItem* = ~items.slice(0, limit ?? ~items.count());
	nextItemId : ()                   => TodoItemId = TodoItemId(1 + ~items.keys()); //TODO
};
```