### angularjs 字符串截取 filter，用于截取字符串

```JavaScript

angular.module('ng').filter('cut', function () {
        return function (value, wordwise, max, tail) {
            if (!value) return '';

            max = parseInt(max, 10);
            if (!max) return value;
            if (value.length <= max) return value;

            value = value.substr(0, max);
            if (wordwise) {
                var lastspace = value.lastIndexOf(' ');
                if (lastspace != -1) {
                    value = value.substr(0, lastspace);
                }
            }

            return value + (tail || ' …');
        };
    });
```

### Usage:

{{some_text | cut:true:100:' ...'}}

### Options:

* wordwise (boolean) - if true, cut only by words bounds,

* max (integer) - max length of the text, cut to this number of chars,

* tail (string, default: ' …') - add this string to the input string if the string was cut.