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

## Add Global State with Zustand

```

import { StoreApi, UseBoundStore, create } from 'zustand'
import { flatten } from 'flatten-anything'
import { FlattenType } from './globalStateHelper'

const _GlobalState = {
    test: {
        test: '',
    },
}
type State = FlattenType<typeof _GlobalState>
const GlobalState = flatten(_GlobalState) as State

const createSetter = (set: any) => (key: keyof State, value: any) => {
    set((state: any) => ({ ...state, [key]: value }))
}

interface Store extends State {
    set: (key: keyof State, value: any) => void
    get: <T extends keyof State>(key: T) => State[T]
    subscribe: (key: keyof State, callback: (value: any) => void) => () => void
}

const basicGlobalState = create<Store>((set, get, api) => ({
    ...GlobalState,
    set: createSetter(set),
    get: (key) => get()[key],
    subscribe: (key, callback) => {
        const listener = (state: State, prevState: State) => {
            if (state[key] === prevState[key]) return
            callback(state[key])
        }
        return api.subscribe(listener)
    },
}))

type ExtendedStore = Omit<UseBoundStore<StoreApi<Store>>, 'subscribe'> & {
    (): Store
    set: <K extends keyof State>(key: K, value: State[K]) => void
    get: <T extends keyof State>(key: T) => State[T]
    subscribe: <K extends keyof State>(
        key: K,
        callback: (value: State[K]) => any
    ) => void
    oldSubscribe: (
        listener: (state: Store, prevState: Store) => void
    ) => () => void
}

const useGlobalState = basicGlobalState as any as ExtendedStore
useGlobalState.oldSubscribe = basicGlobalState.subscribe

useGlobalState.set = <K extends keyof State>(key: K, value: State[K]) =>
    useGlobalState.setState({ [key]: value })
useGlobalState.get = <T extends keyof State>(key: T) =>
    useGlobalState.getState()[key]
useGlobalState.subscribe = <K extends keyof State>(
    key: K,
    callback: (value: State[K]) => any
) => {
    useGlobalState.oldSubscribe((state, prevState) => {
        if (state[key] === prevState[key]) return
        callback(state[key])
    })
}

export default useGlobalState



```


```


type Primitive = string | number | boolean | symbol | undefined | null

type IsArray<T> = T extends Array<any> ? T : never

// Updated helper type to accumulate flattened paths, with special handling for arrays
type Flatten<T, Path extends string = ''> = {
    [K in keyof T]: T[K] extends Primitive
        ? { [P in `${Path}${K & string}`]: T[K] }
        : T[K] extends Array<any>
        ? { [P in `${Path}${K & string}`]: IsArray<T[K]> } // Preserve array type as is
        : Flatten<T[K], `${Path}${K & string}.`>
}[keyof T]

// Utility type to merge all properties into a single type
type Merge<T> = (T extends any ? (x: T) => void : never) extends (
    x: infer R
) => void
    ? R
    : never

// Final Flatten type that applies Merge to consolidate all properties
export type FlattenType<T> = Merge<Flatten<T>>




```
