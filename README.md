# template

It's able to template strings or objects and fill them from a context.

## Example with string

You can use an string as template.

```javascript
const { template } = require('@agarimo/template');

const result = template(`Hello {{ name }} {{ age }}`, { name: 'Anna', age: 30 });
console.log(result); 
// Hello Anna 30
```

Or you can compile the template and get a function that you can use with different contexts.

```javascript
const { compile } = require('@agarimo/template');

const compiled = compile(`Hello {{ name }} {{ age }}`);
const result = compiled({ name: 'Anna', age: 30 });
console.log(result); 
// Hello Anna 30
```

## Example with an object

You can use an object as template.

```javascript
const { template } = require('@agarimo/template');

const source = {
  name: '{{ name }}',
  age: '{{ age }}',
}

const result = template(source, { name: 'Anna', age: 30 });
console.log(result);
// { name: 'Anna', age: '30' }
```

Or you can compile the template and get a function that you can use with different contexts.

```javascript
const { compile } = require('@agarimo/template');

const source = {
  name: '{{ name }}',
  age: '{{ age }}',
}

const compiled = compile(source);
const result = compiled({ name: 'Anna', age: 30 });
console.log(result); 
// { name: 'Anna', age: '30' }
```

## Example with forEach in string

You can use {{ #collection }} to start a forEach over collection, and {{ /# }} to end the forEach.

```javascript
const { template } = require('@agarimo/template');

const context = {
  name: 'Products',
  category: 'software',
  products: [
    { code: 'SO1', name: 'Software 1', value: 500, parts: ['a', 'b', 'c'] },
    { code: 'SO2', name: 'Software 2', value: 600, parts: ['d', 'e', 'f'] },
    { code: 'SO3', name: 'Software 3', value: 700, parts: ['g', 'h', 'i'] },
  ]
}

const input = `For {{ name }} the category is {{ category }}. We have {{ products.length }} products that are: {{#products}}
- {{ code }}: {{ name }} ({{ value }}€) Composed by: {{ parts.join(', ')}}{{ /# }}`;

console.log(template(input, context));
// For Products the category is software. We have 3 products that are:
// - SO1: Software 1 (500€) Composed by: a, b, c
// - SO2: Software 2 (600€) Composed by: d, e, f
// - SO3: Software 3 (700€) Composed by: g, h, i
```

The collection can be a javascript expression. Example, you can use filter

```javascript
const { template } = require('@agarimo/template');

const context = {
  name: 'Products',
  category: 'software',
  products: [
    { code: 'SO1', name: 'Software 1', value: 500, parts: ['a', 'b', 'c'] },
    { code: 'SO2', name: 'Software 2', value: 600, parts: ['d', 'e', 'f'] },
    { code: 'SO3', name: 'Software 3', value: 700, parts: ['g', 'h', 'i'] },
  ]
}

const input = `For {{ name }} the category is {{ category }}. We have {{ products.length }} products that are: {{#products.filter(product => product.value > 500)}}
- {{ code }}: {{ name }} ({{ value }}€) Composed by: {{ parts.join(', ')}}{{ /# }}`;

console.log(template(input, context));
// For Products the category is software. We have 3 products that are:
// - SO2: Software 2 (600€) Composed by: d, e, f
// - SO3: Software 3 (700€) Composed by: g, h, i
```

## Example with iterator in object

If you have an array in an object, each element that contains the field _iterator_ between underscores with a value that starts by #, then it will be consider an iterable object, like in the previous forEach example.

```javascript
const { template } = require('@agarimo/template');

const source = {
  name: '{{ name }}',
  category: '{{ category }}',
  cards: [
    {
      _iterator_: '#products',
      name: '{{ name }}',
      value: '{{ value }}',
    },
  ],
};

const context = {
  name: 'Products',
  category: 'software',
  products: [
    { code: 'SO1', name: 'Software 1', value: 500, parts: ['a', 'b', 'c'] },
    { code: 'SO2', name: 'Software 2', value: 600, parts: ['d', 'e', 'f'] },
    { code: 'SO3', name: 'Software 3', value: 700, parts: ['g', 'h', 'i'] },
  ],
};

console.log(template(source, context));
// {
//   name: 'Products',
//   category: 'software',
//   cards: [
//     { name: 'Software 1', value: '500' },
//     { name: 'Software 2', value: '600' },
//     { name: 'Software 3', value: '700' }
//   ]
// }
```

## Access the current element when iterating

Sometimes you want to access the current element. Also sometimes the elements are not objects, so you will want to access their values directly.
You can do that with _current_ between underscores.

```javascript
const { template } = require('@agarimo/template');

const source = 'Product list: {{ #products }}\n- Product "{{ _current_ }}"{{ /# }}'

const context = {
  name: 'Products',
  category: 'software',
  products: ["SO1", "SO2", "SO3"]
};

console.log(template(source, context));
// Product list:
// - Product "SO1"
// - Product "SO2"
// - Product "SO3"
```

## Access the parent when iterating

You can access the parent object with _parent_ between underscores.

```javascript
const { template } = require('@agarimo/template');

const source = 'Product list: {{ #products }}\n- Product "{{ _current_ }}" category is {{ _parent_.category }}{{ /# }}'

const context = {
  name: 'Products',
  category: 'software',
  products: ["SO1", "SO2", "SO3"]
};

console.log(template(source, context));
// Product list:
// - Product "SO1" category is software
// - Product "SO2" category is software
// - Product "SO3" category is software
```