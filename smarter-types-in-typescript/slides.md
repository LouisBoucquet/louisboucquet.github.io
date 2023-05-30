---
class: invert
title: Smarter types in typescript
---
# Smarter types in typescript

Help typescript help you

![Help me help you meme](https://media.giphy.com/media/fdLR6LGwAiVNhGQNvf/giphy.gif)

---

# What are types?

---

# Types are an abstraction

---

# Types as more than an abstraction

Types can describe your code.

---

# Examples

---

# Selected Item

```ts
type SelectedItem = {
	item?: Item;
	subItem?: SubItem;
};
```

---

# Selected Item

```ts
const item = {
};

const item = {
	item: Item,
};

const item = {
	item: Item,
	subItem: SubItem,
};

const item = {
	subItem: SubItem,
};
```

---

# Selected Item

```ts
type SelectedItem =
	| {}
	| {
			item: Item;
			subItem?: SubItem;
	  };
```

Excluding

```ts
const selectedItem = {
	subItem: SubItem,
} satisfies SelectedItem;
```
