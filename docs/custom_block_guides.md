# Custom Block Guides

## Create Custom Block
Here is an example how to create a custom block:
```php
use pocketmine\block\BlockBreakInfo;
use pocketmine\block\BlockIdentifier;
use pocketmine\block\Opaque;
use pocketmine\item\Item;

class CustomBlock extends Opaque{
    public function __construct(string $name, BlockTypeInfo $typeInfo)
    {
        parent::__construct(new BlockIdentifier(BlockTypeIds::newId()), $name, $typeInfo);
    }
}
```

## Basics
Registering a custom block can either be done with or without a model using the `CustomiesBlockFactory` class. Without a model all you need to do is register the block by providing a `Closure` and the identifier of the block. The closure must be of the type `Closure(int): Block`, where the int is the ID that should be used for the block. This **DOES NOT** allow you to use a custom ID for the block, and will cause issues if you do not use the provided value.

```php
use customiesdevs\customies\block\CustomiesBlockFactory;
use pocketmine\block\BlockBreakInfo;

// ...

public function onEnable(): void {
        $hardeness = 0.3;
        $blockToolType = BlockToolType::PICKAXE;
        $blastResistance = 0.1;
        $creativeInfo = new CreativeInventoryInfo(CreativeInventoryInfo::CATEGORY_CONSTRUCTION, CreativeInventoryInfo::NONE);
        $identifier = "customies:your_custom_block";
        CustomiesBlockFactory::getInstance()->registerBlock(static fn () => new CustomBlock("your_name_here", new BlockTypeInfo(new BlockBreakInfo($hardeness, $blockToolType, 0, $blastResistance))), $identifier, null, $creativeInfo);
}
// ...
```
> It is important to note that the provided closure is shared across threads, meaning you cannot use any variables that are not serializable types defined outside the scope of the closure.

## Getting a Block
To get a custom block you need to use the `CustomiesBlockFactory` instead of the regular block factory so you can get the block from the custom identifier instead of a numeric id.
```php
$block = CustomiesBlockFactory::getInstance()->get("customies:example_block");
```

## Using Custom Models
If your block contains a different model, you can provide a `Model` as the 5th argument. A model requires the following:
- Materials: Array of materials that define how the texture is applied to specific faces
- Texture: Name of the texture to apply to the model
- Origin: The origin point of the selection box. `Vector3(0, 0, 0)` is the top right corner of the block
- Size: The size of the block in pixels. This must be between `Vector3(0, 0, 0)` and `Vector3(16, 16, 16)` as the client
  does not support blocks being larger than this

```php
use customiesdevs\customies\block\CustomiesBlockFactory;
use customiesdevs\customies\block\Material;
use customiesdevs\customies\block\Model;
use pocketmine\block\BlockBreakInfo;
use pocketmine\math\Vector3;

// ...

public function onEnable(): void {
	$material = new Material(Material::TARGET_ALL, "example", Material::RENDER_METHOD_ALPHA_TEST);
	$model = new Model([$material], "geometry.example", new Vector3(-8, 0, -8), new Vector3(16, 16, 16));
    CustomiesBlockFactory::getInstance()->registerBlock(static fn () => new CustomBlock("your_name_here", new BlockTypeInfo(new BlockBreakInfo(0.3, BlockToolType::PICKAXE, 0, 0.1))), "customies:your_custom_block", $model, new CreativeInventoryInfo(CreativeInventoryInfo::CATEGORY_CONSTRUCTION, CreativeInventoryInfo::NONE));
}

// ...
```

> More information about materials and the different properties can be found on [docs.microsoft.com](https://docs.microsoft.com/en-us/minecraft/creator/reference/content/blockreference).

## Creative Inventory
Try reading [Item Creative Inventory](/custom_item#creative-inventory) it is the same concept.

## Updating from v1.0.7 -> newer versions
In Customise v1.1.0, the method for registering custom blocks changed to be more dynamic and user friendly. Instead of providing the name and `BlockBreakInfo` to the `registerBlock()` method, you are now required to provide a `Closure` which returns a `Block` with the provided ID. Below is an example difference between the two versions for registering a custom block.
```diff
use customiesdevs\customies\block\CustomiesBlockFactory;
use pocketmine\block\BlockBreakInfo;

// ...

public function onEnable(): void {
-	CustomiesBlockFactory::getInstance()->registerBlock(Block::class, "customies:example_block", "Example Block", new BlockBreakInfo(1));
+	CustomiesBlockFactory::getInstance()->registerBlock(static fn () => new CustomBlock("your_name_here", new BlockTypeInfo(new BlockBreakInfo(0.3, BlockToolType::PICKAXE, 0, 0.1))), "customies:your_custom_block"", null, new CreativeInventoryInfo(CreativeInventoryInfo::CATEGORY_CONSTRUCTION, CreativeInventoryInfo::NONE));

}

// ...
```
> You can learn more about this change and why it was necessary [here](https://github.com/CustomiesDevs/Customies/pull/37).