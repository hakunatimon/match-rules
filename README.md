# matchRules

matchRules is a javascript utility for matching an object to defined conditions on the properties of that object (aka Rules). It can be used on feature flags, complex conditions, conditional rendering, etc.

### Install

Through npm
`npm install matchrules --save`
Through Yarn
`yarn add matchrules`

### Usage

ES6

```js
import { matchRules } from "matchrules";
```

ES5

```js
const matchRules = require("matchrules").matchRules;
```

TypeScript

```js
import { matchRules } from "matchrules";
```

### API

```js
// returns a boolean value.
matchRules(
  sourceObject, // can be any object with data.
  RULES_OBJECT, // you can also pass multiple rules in an array [RULE_ONE, RULE_TWO],
  options, // (optional)
);

const options = {
  operator: 'and', // (optional, default: 'and') in case of multiple rules you can pass 'and' or 'or'. In case of 'or' your rules will be compared with 'or' operator. Default is 'and'
  debug: true, // (optional, default: false) when debug is true, it logs a trace object which will tell you which rule failed and with what values of source and rules object.
},

// NOTE: all the rules inside a single rule are concatinated by 'and' operator.
```

### Example (Usecase)

```js
// rules object
import { matchRules } from "matchrules";

const SHOW_JOB_RULE = {
  hasVisa: true,
  profile: {
    country: "US",
    yearsOfExperience: (exp, sourceObject) => exp > 3,
  },
};

// source object
const user = {
  username: "someName",
  hasVisa: true,
  profile: {
    country: "US",
    yearsOfExperience: 5,
    yearOfGraduation: 2011,
  },
};

// pass source and rules
if (matchRules(user, SHOW_JOB_RULE)) {
  //... do something conditionally.
}
```

### Features

- **Multiple rules support** - you can pass multiple rules dealing with a common source object.

- **Graceful exit** - returns false if the asked property in the rule doesn’t exist in the source object.

- **Debugging** - when enabled logs a trace object for all the keys in the rule with a meaningful message of what went wrong.

- **Function support** - you can pass custom functions in the rule for handling complex conditions.

- **Nested Rules** - you can pass a rule no matter how deep your data source is.

- **Multiple operator support** - you can pass or / and operators in case of multiple rules.
  Dillinger is a cloud-enabled, mobile-ready, offline-storage, AngularJS powered HTML5 Markdown editor.

### Function Support:

So far it is capable to handle any condition since you can write your own functions in the rule.

when it encounters a function it passes the value from the source and original source object to that function matching the corresponding key of that level.

For example:

```js
const SHOW_ADS_RULES = {
  profile: {
    age: (value, sourceObject) => value > 18 && value < 55,
  },
};

const source = {
  profile: {
    age: 20,
  },
};

// so value of 20 and whole source object will be passed to that function.
// NOTE: you should always return boolean value from the function you implement.
```

### Extend Rules (avoid redundancy)

```js
const SHOW_ADS_RULES = {
  onboarding: true,
  admin: false,
  profile: {
    country: "US",
    school: "MIT",
    age: (value) => value > 18 && value < 55,
  },
};

// show a different Ad if the country is India.
const SHOW_ADS_RULES_INDIA = {
  ...SHOW_ADS_RULES,
  profile: {
    ...SHOW_ADS_RULES.profile,
    country: "India",
  },
};
```

### More examples

**Ex 1. Feature Flags**

```js
import { matchRules } from 'matchrules';

// this object can come from your app state
const sourceObject = {
    enable_unique_feature = true,
};

// Rule
const ENABLE_UNIQUE_FEATURE = {
    enable_unique_feature: true,
};

if(matchRules(sourceObject, ENABLE_UNIQUE_FEATURE)) {
    // render unique feature
}
```

**Ex 2. Multiple Rules and functions implementation**

```js
import { matchRules } from 'matchrules';

// this object can come from your app state
const sourceObject = {
    enable_unique_feature = true,
    profile: {
        age: 18,
    },
};

// Rules
const ENABLE_UNIQUE_FEATURE = {
    enable_unique_feature: true,
};

const ENABLE_UNIQUE_FEATURE_WITH_AGE_18YO = {
    profile: {
        age: (value, sourceObject) => value > 18,
    },
};

// by default multiple rules will be combined using AND operator
if(matchRules(sourceObject, [ENABLE_UNIQUE_FEATURE, ENABLE_UNIQUE_FEATURE_WITH_AGE_18YO])) {
    // render unique feature
}
```

**Ex 3. Multiple Rules using OR operator**

```js
import { matchRules } from 'matchrules';

// this object can come from your app state
const sourceObject = {
    enable_unique_feature = true,
    profile: {
        age: 18,
        country: 'US',
    },
};

// Rules
const ENABLE_UNIQUE_FEATURE_FOR_US = {
    profile: {
        country: 'US',
    },
};

const ENABLE_UNIQUE_FEATURE_FOR_INDIA = {
    profile: {
        country: 'IN',
    },
};

// to combine rules using OR, (display feature if user is from US or INDIA)
if(matchRules(
    sourceObject,
    [ENABLE_UNIQUE_FEATURE_FOR_US, ENABLE_UNIQUE_FEATURE_FOR_INDIA],
    { operator: 'or'})) {
    // render unique feature
}

// you can pass as many rules you want
```

**Example 3 using functions**

```js
import { matchRules } from 'matchrules';

// this object can come from your app state
const sourceObject = {
    enable_unique_feature = true,
    profile: {
        age: 18,
        country: 'US',
    },
};

// Rules
const ENABLE_UNIQUE_FEATURE_FOR_US_OR_INDIA = {
    profile: {
        country: (value, sourceObject) => value === 'US' || value === 'IN',
    },
};

// to combine rules using OR, (display feature if user is from US or INDIA)
if(matchRules(
    sourceObject,
    ENABLE_UNIQUE_FEATURE_FOR_US_OR_INDIA)) {
    // render unique feature
}

// you can use functions to deal with complex scenarios
```