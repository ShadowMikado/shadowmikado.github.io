# Custom Item Guides

## Registering an Item
Registering a custom item is as simple as registering a normal item, but the ID is calculated for you. All you need to
do is use the `CustomiesItemFactory` class to register the item, and fetch it as you would with a vanilla item.

```php
use customiesdevs\customies\item\CustomiesItemFactory;

// ...

public function onEnable(): void {
	CustomiesItemFactory::getInstance()->registerItem(Item::class, "customies:example_item", "Example Item");
}

// ...
```

## Getting an Item
To get a custom item you need to use the `CustomiesItemFactory` instead of the regular item factory so you can get the item from the custom identifier instead of needing a numeric id.
```php
$item = CustomiesItemFactory::getInstance()->get("customies:example_item");
```

## Item Components
Custom items can also have components which are used to change the behaviour of items client side, such as making it edible or have durability etc. To get started with components, you need to implement the `ItemComponents` interface, use the `ItemComponentsTrait` and call the `initComponent` method in the constructor of your class.

```php
use customiesdevs\customies\item\ItemComponents;
use customiesdevs\customies\item\ItemComponentsTrait;
use pocketmine\item\Item;

class ExampleItem extends Item implements ItemComponents {
	use ItemComponentsTrait;

	public function __construct() {

        parent::__construct(new ItemIdentifier(ItemTypeIds::newId()), "your_name_here", []);
		$this->initComponent("your_texture_here", new CreativeInventoryInfo(CreativeInventoryInfo::CATEGORY_ITEMS));
	}
}
```

Now that you have an item with components, you can add either components or properties using the `addComponent`
and `addProperty` methods (after initializing the item).

```diff
// ...

+	$this->addComponent(new DurabilityComponent(3000))
+	$this->addComponent(new MaxStackSizeComponent(1));
+   $this->addComponent(new AllowOffHandComponent(false));
-   $this->addProperty("allow_off_hand", false)
-   $this->addComponent("minecraft:armor", ["protection" => new IntTag(4)]);

// ...
```

> More information about all the different item components and properties can be found on [docs.microsoft.com](https://docs.microsoft.com/en-us/minecraft/creator/reference/content/itemreference).

## Creative Inventory
If you want add the item to the creative inventory, you can provide a `CreativeInventoryInfo` as the 3th argument to `initComponent`. This contains the following:
 - Category: The base category to show the item in, for example `CreativeInventoryInfo::CATEGORY_EQUIPMENT` will put the item in the equipment category
 - Group: The group to put the item in inside of the category, for example `CreativeInventoryInfo::GROUP_SWORD` will put the item at the end of the sword group

```php
use customiesdevs\customies\item\CreativeInventoryInfo;

// ...

$creativeInfo = new CreativeInventoryInfo(CreativeInventoryInfo::CATEGORY_EQUIPMENT, CreativeInventoryInfo::GROUP_SWORD);
$this->initComponent("your_texture_here", $creativeInfo);

// ...
```

> More information about creative categories and groups can be found on [docs.microsoft.com](https://docs.microsoft.com/en-us/minecraft/creator/reference/content/blockreference/examples/blockcomponents/minecraftblock_creative_category).

## Regulating Held Item Scale
If you are using a custom item that has a texture larger than 16x16 pixels, you may notice the scale of the item also increases when held in a player's hand. To solve this you can use the `setupRenderOffsets()` method inside the `ItemComponentsTrait`. This requires the `initComponent()` method to first be called. The method accepts two integers for the width and height of the image, and an optional bool for if the item is hand equipped, e.g. a sword/tool.
```php
use customiesdevs\customies\item\ItemComponents;
use customiesdevs\customies\item\ItemComponentsTrait;
use pocketmine\item\Item;

class ExampleItem extends Item implements ItemComponents {
	use ItemComponentsTrait;

	public function __construct(ItemIdentifier $idInfo) {

		parent::__construct($idInfo);

		$this->initComponent("your_texture_here");
		$this->setupRenderOffsets(32, 32, true);
	}
}
```
