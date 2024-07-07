# Block Permutations

> It is important to note that permutations are a very complex system, at least compared to the rest of the Customies
> API. For support with the Customies API, you can [join our Discord](https://discord.gg/Tm6wGxWqgh).

## What are permutations?

In Minecraft, permutations are a way for a single block to behave different based on server-side data. Without
permutations, a block can only have one possible state and behave similar to a dirt block. With permutations however,
you can change the model or rotation of the same block to emulate the behaviour of a crop or a door etc.

## How do they work?

When creating a block with permutations, there are two important sets of data that you must provide. The first is an
array of block properties. Each block can have multiple block properties, and each one has a name and an array of all
its possible values. The second set of data is an array of permutations which uses the block properties in molang
queries to provide different block components for each different state. Both of these are sent to the client so it knows
how to render every possible combination of the block.

## Rotating Blocks

Customies includes a built-in `RotatableTrait` which can be used to implement horizontal rotations for your block. You
do not need to write any extra code for it to function, and can easily be followed as an example of how permutations
should be created.

```php
use customiesdevs\customies\block\permutations\Permutable;
use customiesdevs\customies\block\permutations\RotatableTrait;
use pocketmine\block\Opaque;

class ExampleBlock extends Block implements Permutable {
	use RotatableTrait;
}
```

## Creating Your Own Permutations

To get started with adding permutations to your block, you must first implement the `Permutable` interface. This
interface requires you to implement all the methods necessary for permutations to function for a block. To help
understand how permutations work, code from `RotatableTrait` will be used as examples.

### `getBlockProperties()`

In this method you must return an array of `BlockProperty` objects which can be used to determine the state of the
block. In the example below, it is creating a property called `customies:rotation` with the possible values
of `[2, 3, 4, 5]`. These numbers are the values from `Facing::NORTH`, `Facing::SOUTH`, `Facing::WEST` & `Facing::EAST`.
The values do not need to be anything specific, and can use any primitive data type.

```php
public function getBlockProperties(): array {
	return [
		new BlockProperty("customies:rotation", [2, 3, 4, 5]),
	];
}
```

### `getPermutations()`

In this method you must return an array of `Permutation` objects which can override different components based on a
condition of the state. In the example below, there are four different permutations which all check for a different
value of the `customies:rotation` value that was defined in the returned block properties. Each permutation also
contains a single `minecraft:rotation` component which rotates the block to the correct direction for the state. The
condition string accepts molang expressions, as well
as [most entity queries](https://bedrock.dev/docs/stable/Molang#List%20of%20Entity%20Queries). The different block
components can also seen on [bedrock.dev](https://bedrock.dev/docs/stable/Blocks#Block%20Components).

```php
public function getPermutations(): array {
	return [
		(new Permutation("q.block_property('customies:rotation') == 2"))
			->withComponent("minecraft:rotation", CompoundTag::create()
				->setFloat("x", 0)
				->setFloat("y", 0)
				->setFloat("z", 0)),
		(new Permutation("q.block_property('customies:rotation') == 3"))
			->withComponent("minecraft:rotation", CompoundTag::create()
				->setFloat("x", 0)
				->setFloat("y", 180)
				->setFloat("z", 0)),
		(new Permutation("q.block_property('customies:rotation') == 4"))
			->withComponent("minecraft:rotation", CompoundTag::create()
				->setFloat("x", 0)
				->setFloat("y", 90)
				->setFloat("z", 0)),
		(new Permutation("q.block_property('customies:rotation') == 5"))
			->withComponent("minecraft:rotation", CompoundTag::create()
				->setFloat("x", 0)
				->setFloat("y", 270)
				->setFloat("z", 0)),
	];
}
```

### `getCurrentBlockProperties()`

In this method you must return an array of the server-side values which match the order and possible values of all the
block properties for the block. In the example below, only the facing property from the trait is returned since it is
the only block property.

If the order of the values is incorrect, or invalid values are provided, runtime errors may occur when registering and
using the blocks.

```php
public function getCurrentBlockProperties(): array {
	return [$this->facing];
}
```

<hr>

After you have implemented the `Permutable` interface, there are still a few more steps that you must complete before
your block is usable. For pocketmine to be able to save and understand the different state, you must override
the `writeStateToMeta()`, `readStateFromData()` & `getStateBitmask()` methods from the `Block` class.

The first method requires translating the block properties in to a numeric meta value which pocketmine can understand.
The `Permutations::toMeta()` method handles this conversion for you.

The second method is used to load the block properties from the numeric meta value that was previously calculated.
The `Permutations::fromMeta()` method handles the conversions for you, but you are still required to set the properties
to the values from the returned array. The order of the returned properties will always be the same as the order of your
defined block properties.

The third and final method returns the highest bit required to represent all possible states of the block.
The `Permutations::getStateBitmask()` method handles this calculation for you.

```php
protected function writeStateToMeta(): int {
	return Permutations::toMeta($this);
}

public function readStateFromData(int $id, int $stateMeta): void {
	$blockProperties = Permutations::fromMeta($this, $stateMeta);
	$this->facing = $blockProperties[0] ?? Facing::NORTH;
}

public function getStateBitmask(): int {
	return Permutations::getStateBitmask($this);
}
```

## Using Permutations

Using your new permutations is just as simple as changing the different values for each block property when appropriate.
In the case of `RotatableTrait`, the facing property is set in the `place()` method of the block and is automatically
placed using the correct rotation. This method is not a requirement, and the properties can be updated from anywhere and
will still work when setting the block in the world.

```php
public function place(BlockTransaction $tx, Item $item, Block $blockReplace, Block $blockClicked, int $face, Vector3 $clickVector, ?Player $player = null): bool {
	if($player !== null) {
		$this->facing = $player->getHorizontalFacing();
	}
	return parent::place($tx, $item, $blockReplace, $blockClicked, $face, $clickVector, $player);
}
```