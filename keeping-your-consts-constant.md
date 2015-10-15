The only way to ensure you're still importing a read only value ([`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)) with es2015's new module system is to explicitly import the value.

##### _consts.js_

```javascript
export const VERSION = '1.0.0'
export const PORT = 3000
```

##### _index.js_

```javascript
import {VERSION, PORT} from './consts'

VERSION = '1.1.0'
// this won't work as the value imported is still a const
```

If you were to import any other way, you're creating a brand new object and assigning all the exported values to it, thus losing their read onlyness.

##### _index.js_

```javascript
import * as consts from './consts'

consts.VERSION = '1.1.0'
// unlike the example above this will work as the value is now an object property
```

This pattern results in a cleaner import but you're losing the read only benefits of defining the const in the first place.

You're also leaving the value open to be amended, either accidently or not, at a later point which could result in unexpected behaviour and wasted time debugging.
