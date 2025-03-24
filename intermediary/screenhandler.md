# ScreenHandler
### Para o que serve?
**ScreenHandler** é uma classe que serve para sincronizar os conteúdos de um inventário entre client e servidor. Geralmente o conteúdo vem do bloco pai, como uma fornalha ou báu, mas você pode criar um inventário novo on demand para "inventários virtuais".

É dentro do **ScreenHandler** que você define a posição dos **Slots** do inventário, comportamento de quando ele for fechado, comportamento do quickMove (shift + click) e se ele pode ser utilizado.

**ScreenHandler** precisa de um **syncId: Int** e de um **ScreenHandlerType**, que é composto de um Identifier para lambda com a referência da nossa classe **ScreenHandler** e o inventário do jogador.
___
### Exemplo
```kotlin
// NOMEAR COM O NOME DA GUI "XScreenHandler"
// ModScreensHandlers.CUSTOM_SCREEN_HANDLER NÃO EXISTE AINDA
// E SERÁ CRIADO EM BAIXO.
class CustomScreenHandler(syncId: Int, playerInventory: PlayerInventory): ScreenHandler(ModScreenHandlers.CUSTOM_SCREEN_HANDLER, syncId) {
    // INVENTÁRIO ON DEMAND
    private val inventory = SimpleInventory(6)

    init {
        // SLOTS DO INVENTÁRIO ON DEMAND
        for (i in 0 until 6) {
            addSlot(Slot(inventory, i, 44 + (i % 3) * 18, 18 + (i / 3) * 18))
        }

        // SLOTS DO INVENTÁRIO PRINCIPAL
        for (y in 0 until 3) {
            for (x in 0 until 9) {
                addSlot(Slot(playerInventory, x + y * 9 + 9, 8 + x * 18, 84 + y * 18))
            }
        }

        // SLOTS DA HOTBAR
        for (x in 0 until 9) {
            addSlot(Slot(playerInventory, x, 8 + x * 18, 142))
        }
    }

    // LÓGICA DE FECHAR A GUI, COMO É ON DEMAND
    // DEVOLVE O ITEM E REMOVE DA INSTANCIA DO INVENTÁRIO
    override fun onClosed(player: PlayerEntity) {
        super.onClosed(player)

        for (i in 0 until inventory.size()) {
            val stack = inventory.getStack(i)
            if (!stack.isEmpty) {
                player.inventory.offerOrDrop(stack)
                inventory.setStack(i, ItemStack.EMPTY)
            }
        }
    }

    // LÓGICA DE SHIFT + CLICK
    override fun quickMove(player: PlayerEntity, index: Int): ItemStack {
        val slot = slots[index]
        if (!slot.hasStack()) return ItemStack.EMPTY
        val stack = slot.stack
        val originalStack = stack.copy()
        if (index < 6) { // MOVE DA GUI PARA O INVENTÁRIO
            if (!insertItem(stack, 6, slots.size, true)) return ItemStack.EMPTY
        } else { // MOVE DO INVENTÁRIO PARA A GUI
            if (!insertItem(stack, 0, 6, false)) return ItemStack.EMPTY
        }
        if (stack.isEmpty) {
            slot.stack = ItemStack.EMPTY
        } else {
            // ATUALIZA O INVENTÁRIO DO PLAYER
            slot.markDirty()
        }
        return originalStack
    }

    // SE PODE SER UTILIZADO
    override fun canUse(player: PlayerEntity?): Boolean {
        return true
    }
}
```

```kotlin
object ModScreenHandlers {
	val CUSTOM_SCREEN_HANDLER: ScreenHandlerType<CustomScreenHandler> = ScreenHandlerRegistry.registerSimple(
		Identifier("modid",
		"custom_gui")
	) { syncId, inv ->
		CustomScreenHandler(syncId, inv)
	}
	fun register() {}
}
```