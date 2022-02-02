# CheatSheet

## Return new object of modified property using (setProperty) and defining property like (property)

```
return new Proxy(this, {
  get(target, name: string, receiver) {
    if (name.startsWith('set')) {
      const setName = name.substring(3);
      const definedName = setName.charAt(0).toLowerCase() + setName.slice(1);
      const newLocalState = Object.assign(new LocalState(), target) as any;
      return (newVal: any) => {
        newLocalState[definedName] = newVal;
        return newLocalState;
      };
    }
    return (target as any)[name];
  },
});
```
