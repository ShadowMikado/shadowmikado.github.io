# Custom Entity Guides

## Custom Entity Class
Before you can register an entity, you need to create a class extending `Entity` that uses your custom identifier. This is required for pocketmine to know the type of entity to spawn.

```php
use pocketmine\entity\Entity;

class ExampleEntity extends Entity {

	// ...

	public static function getNetworkTypeId(): string {
		return "customies:example_entity";
	}
	
	// ...
}
```

## Registering an Entity
Registering a custom entity is as simple as registering a normal entity. All you need to do is use the `CustomiesEntityFactory` class to register the entity and use the same identifier from the previous step.

```php
use customiesdevs\customies\entity\CustomiesEntityFactory;

// ...

public function onEnable(): void {
	CustomiesEntityFactory::getInstance()->registerEntity(ExampleEntity::class, "customies:example_entity");
}

// ...
```

> If you want to provide your own creation function, you can provide a `Closure(World $world, CompoundTag $nbt): Entity` as the 3rd argument.

## Creating an Entity
Creating a custom entity is identical to how you would do it normally with any other entity, and the same applies for spawning it.
```php
$entity = new ExampleEntity(...);
$entity->spawnToAll();
```