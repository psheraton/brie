This is barry
=============
This Business Rules Engine (B.R.E, or "barry") is a transient Feature Flipping Criteria System for Node, based on [fflip](https://github.com/FredKSchott/fflip).

```
npm install ssh://git@bitbucket01.corp.sea1.cmates.com:7999/siteui/barry.git
```

## Getting Started
Below is a simple example that uses __barry__ to deliver feature flags based on a determined (set) of User Variant(s):
```javascript
// Include barry
var barry = require('barry');

// given an inbound data object called "user" plus a set of features (see below) called "featureSet"
barry.setup(
  {
    data: user,
    features: featureSet
  }
);
var flags = barry.getAll();
// expect {feature1: true, feature2: false, ... }
```

### Criteria
Criteria are the predefined rule types that __barry__ tests data against, per feature.  Each feature in the feature set consists of at least one criteria.  A feature may contain multiple criteria, joined across a logical "and" or "any" as dictated by the `criteriaLogic`:
```javascript
  var featureSet = multiPartTestCase : {
    criteria : [
      {
        has: {
          "trait": "messageCount",
          "comparison": "above",
          "value": 2
        }
      },
      {
        has: {
          "trait": "creationDate",
          "comparison": "older",
          "value": "12/Dec/2000"
        }
      }
    ],
    criteriaLogic: "any"
  };
// expect {multiPartTestCase: true}
```


__barry__ has:


#### allowIDs (object, array)
Provided an object containing an "id" property, barry evaluates the presence of the id in the given array, checking for 1 or more entries.

#### always(object, bool)
`Always` asks barry to "always" respond with the given input.  Why?  Code consistency, mainly.

#### percentScale(object, opts)
```
  opts = {
    percentMin:[0-1],
    percentMax:[0-1],
    salt:[number],
    testPhase:[string]
  }
```
"testPhase" is used for logging and debugging only, and does not impact the algorithm.

The `object.id` is used to calculate a percentage, modified by the salt value. If the resulting, salted, number is within range, returns true; otherwise false.

#### has(test data [object], comparison data [object])
The most complex criteria mechanism, `has` will evaluate the test data (first argument) against the trait, comparator and value provided in the second argument.  If the test data has the trait and the associated value evaluates properly considering the comparison value, then true is returned.

* Requires a data object
* Requires a set of comparison data containing the following properties:
  * trait (required) - the name of a property to find on data object.  If comparison and value are omitted, the evaluation will simply verify the existence of the property on the object.
  * comparison (optional) - one of an existing set of comparison instructions :
    * equals (string, date, number, object)
    * like (string)
    * below (string, date, number, object)
    * above (string, date, number, object)
    * longer (string, date, number, object)
    * shorter (string, date, number, object)
    * older (date)
    * younger (date)
  * value (optional) - static value for comparison

If the comparison object contains only `trait` then barry will evaluate the presence of the property on the data object. Comparison and value must be provided together, when one of them is provided.

**Note:** for the purpose of comparisons, "object" refers to both `object` and `array`, with the distinction being left up to the return value of the [_.isArray() method](https://lodash.com/docs#isArray).

Possible comparisons are:

##### equals
 * (`date`) - a **strict** equality check between base comparison date and check date.  If the check value is a number, then barry evaluates the difference between `now()` and the comparison date to be equal to the check value, in days (e.g. "0" implies "today").
 * (`numeric`) - a **non-strict** equality check.
 * (`object`) - a **deep comparison** equality check, using the [_.isEqual()](https://lodash.com/docs#isEqual) method from `lodash-node`.
 * (`string`) - a **strict** equality check that the comparison string is identical to the check value note that this is a case-sensitive check.  See "like", below.

##### above
 * (`date`) - [alias: "longer", "older"] checks to see that the comparison date is older than the check date.  If the check value is a number, barry checks to see that the comparison date is at least as old, in days, as the check value, compared to `now()` (e.g. "1" implies the date must be yesterday or older).
 * (`numeric`) - [alias: "longer"] a greater-than-or-equal-to check that the comparison number is greater than the check value.
 * (`object`) - an array or object is said to be "above" another if it fully contains the other.  That is: if the data object contains the comparison object, then the data object is above the comparison object.  If the objects are equal, the `above` comparison is inherently false.  If the data object is a non-array object but the comparison object is an array, then the [_.difference()](https://lodash.com/docs#difference) comparison is done between the keys of the data object and the comparison array.  If both data and comparison objects are arrays, a lodash [_.difference()](https://lodash.com/docs#difference) between comparison and data is compare with `[]`, indicating that the data object fully contains the comparison object.
 * (`string`) - performs a javascript string "greater-than-or-equal" comparison.  As a loosely-typed language, this could be anything, really.

##### below
 * (`date`) - [alias: "shorter", "younger"] checks to see that the comparison date is more recent than the check date.  If the check value is a number, barry checks to see that the comparison date is at least as new, in days, as the check value, compared to `now()` (e.g. "1" implies the date must be tomorrow or younger).
 * (`numeric`) - [alias: "shorter"] a less-than-or-equal-to check that the comparison number is smaller than the check value.
 * (`object`) - an array or object is said to be "below" another if it is fully contained in the other.  That is: if the comparison object contains the data object, then the data object is below the comparison object.  If the objects are equal, the `below` comparison is inherently false.  If the data object is a non-array object but the comparison object is an array, then the [_.difference()](https://lodash.com/docs#difference) comparison is done between the keys of the data object and the comparison array.  If both data and comparison objects are arrays, a lodash [_.difference()](https://lodash.com/docs#difference) between comparison and data is compare with `[]`, indicating that the data object fully contains the comparison object.
 * (`string`) - performs a javascript string "less-than-or-equal" comparison.  As a loosely-typed language, this could be anything, really.

##### like
 * (`string`) - a **non-strict** check that the `toLowerCase()` comparison string is equal to the `toString().toLowerCase()` check value.

##### longer
 * (`object`) - true if the [_.size()](https://lodash.com/docs#size) of the data object is greater than that of the comparison object.
 * (`string`) - true if the length of the comparison string (non-trimmed) is greater than or equal to the length of the check value (non-trimmed).

##### shorter
 * (`object`) - true if the [_.size()](https://lodash.com/docs#size) of the data object is less than that of the comparison object.
 * (`string`) - true if the length of the comparison string (non-trimmed) is less than or equal to the length of the check value (non-trimmed).

#### Logic Chaining


### Features
Features contain sets of criteria to test users against. The value associated with the criteria is passed in as the data argument of the criteria function. A user will have a featured enabled if they match all listed criteria, otherwise the feature is disabled. Features can include other optional properties for context. Features are described as follows:
```javascript
var ExampleFeaturesObject = {
  "canCheckAlways": {
    "criteria": [
      {
        "always": false
      }
    ]
  },
  "canCheckHasString": {
    "criteria": [
      {
        "has": {
          "trait": "hasStringValue",
          "comparison": "equals",
          "value": "a string check value"
        }
      }
    ]
  },
  "canCheckHigherNumber": {
    "criteria": [
      {
        "has": {
          "trait": "hasNumberValue",
          "comparison": "above",
          "value": 1
        }
      }
    ]
  },
  "canCheckLowerDate": {
    "criteria": [
      {
        "has": {
          "trait": "hasDateValue",
          "comparison": "younger",
          "value": new Date(2000, 1, 1, 1, 22, 0)
        }
      }
    ]
  }
}
```

## Usage
```
Object setup(options)         // barry needs to be initialized with a data object and a set of features.  Don't try to make barry do stuff without setting up first. That makes barry angry.
Bool   get(string feature)    // asks barry if the feature is enabled.
Object getAll()               // requests the full evaluation of features from barry
```
### Setup
In the setup (initializer), barry accepts:
* (`object`) - data: a object structure containing the data to be tested.
* (`object`) - features: see feature section, above.
* (`object`) - overrides: containing properties that match feature names, an override is a simple set of boolean boolean values to force a feature into a state.

  ```
    {
      "canCheckAlways": true
    }
  ```
* (`bool`) - showLogs: when true, barry is verbose about the request activities.

Returns `barry`.

Once initialized, via `setup()`, barry can be queried for the outcome of any or all features.

### get
`get` returns the outcome of a single feature, and requires the string name of the feature, as an argument, and is invoked as
```javascript
var isHigherNumber = barry.get('canCheckHigherNumber');
// isHigherNumber = true;
```

### getAll
`getAll` returns a hash of all features and their evaluated value, including overrides, as a shallow javascript object.
```javascript
var allFeatures = barry.getAll();
// allFeatures = {
// canCheckAlways: true,
// canCheckHasString: true,
// canCheckHigherNumber: false,
// canCheckLowerDate: false
// }
```

### Chaining
A single method can be chained against the `setup` method. `setup` returns `barry`, which has both a `get` and `getAll` method.  Further chaining is not available, since `get` returns a boolean and `getAll` returns an object.
```javascript
var allFeatures = barry.setup({
    data: data_in,
    features: flags_in,
    overrides: overrides,
    showLogs: true
}).getAll()
```
