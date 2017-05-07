# Set of input tools for formatting

This project allow to create mask input easily.
In real world you often need to create input for credit card, phone number or birthday date etc. 
Each of this usecases require to input value with some formatting (for example 0000-0000-000-0000 for credit card) and with static length. This project is going to help you.

Watch demo: http://xnimorz.github.io/masked-input/

# Components

## react-maskinput

A react component, that build on top of input-core. Provide interface for creating custom formatting inputs.
If you work on project without react, you may use mask-input.

### Installation

```
npm install --save react-maskinput
```

### Usage

Simple usage:

```javascript
import MaskInput from 'react-maskinput';

ReactDOM.render(
    someElement, 
    <MaskInput
        alwaysShowMask
        maskChar='_'
        mask='0000-0000-0000-0000'        
    />
)
```

Changing mask in runtime, getting value on input change:

```javascript
import React, { Component } from 'react';
import { render } from 'react-dom';
import MaskInput from 'react-maskinput';

class DateInput extends Component {
    state = {
        maskString: 'DD.MM.YYYY',
        mask: '00.00.0000',
    }

    onChange = (e) => {
        // 2 — for example        
        if (parseInt(e.target.value[6],  10) > 2) {
            this.setState({
                maskString: 'DD.MM.YY',
                mask: '00.00.00',
            });
        } else {
            this.setState({
                maskString: 'DD.MM.YYYY',
                mask: '00.00.0000',
            });
        }
    }

    render() {
        return (
            <MaskInput 
                onChange={this.onChange}
                maskString={this.state.maskString}
                mask={this.state.mask}
                alwaysShowMask
            />
        );
    }
}

render(someElement, <DateInput />);
```

Custom formatting function. For custom formatting function you can see react-numberinput component:

```javascript
import React, { Component } from 'react';
import MaskInput from 'react-maskinput';

function removeSelectedRange(value, selection) {        
    if (selection.start === selection.end) {
        return value;
    }

    if (selection.end < selection.start) {
        const tmp = selection.end;
        selection.end = selection.start;
        selection.start = tmp;
    }    
    
    if (value.length > selection.start) {
        return value.slice(0, selection.start).concat(value.slice(selection.end, value.length));
    }
    
    return value;
}

class NumberInput extends Component {
    // In reformat function you need to set value and selection props
    reformat = ({data, input = '', selection}) => {
        const newSelection = {
            start: selection.start,
            end: selection.end,
        };

        let value = removeSelectedRange(data.replace(/(\D)/g, (text) => text === ' ' ? ' ' : ''), newSelection);
        const inputValue = input.replace(/\D/g, '');
        const oldLength = value.length;

        value = value.slice(0, newSelection.start) + inputValue + value.slice(newSelection.start, value.length);        
        value = value.replace(/\s/g, '').replace(/(\d)(?=(\d\d\d)+(?!\d))/g, (text) => `${text} `);

        let index = newSelection.start;
        if (inputValue) {
            index = Math.max(0, value.length - oldLength + index);
        }
        newSelection.end = newSelection.start = index;        

        return {
            value,
            maskedValue: value,
            visibleValue: value,
            selection: newSelection,
        };
    }

    render() {
        return (
            <MaskInput {...this.props} reformat={this.reformat} />
        );
    }
}

export default NumberInput;
```

### Props

List of specific react-maskinput props:

* mask,
* reformat,
* maskFormat,
* maskChar,
* maskString,
* showMask,
* alwaysShowMask,
* getReference,
* onChange

Important! All other props'll passed to input directly.

Let's see what's doing each of props:

`mask`: String. Format:
   0 — any number 0-9
   * — any symbol
   a — A-Z, a-z
   q — "q" letter, 2 — "2" letter etc.
   \a — "a" letter 
 default is undefined 

 [function] `reformat`: user function, if you want use custom reformat logic. It's userfull for numeric inputs, decimal numbers etc. 
 If reformat defined mask'll be ignored. Reformat function must receive object with several fields:
 ```javascript
 function reformat({data: data, selection: {start, end}, input}) {
     // realisation
 
     return {
         [any] value: // value that store and calling in input core funcitons (such as reformat). value may have any format,
         [String] visibleValue: // value that displayed to user in input if showMask is false,
         [String] maskedValue: // value that  displayed to user in input if showMask is true,
         [{[integer] start, [integer] end}] selection: {start, end} — // new selection range
     }
 }
 ```
 
 if `reformat` and `mask` is undefined, input allow to enter any values. 
 
 You can define custom mask by passing `maskFormat`. This prop must be an array,
 each object in array have several fields:
 `str`: matched char for mask
 `regexp`: validation rule as regexp
 `type`: special  

 `maskChar`: Character to cover unfilled editable parts of mask. Default value is ''.
 `maskString`: String to cover unfilled editable parts of mask. Default is undefined. If maskString define maskChar ignored.

 `showMask`: show mask in input. It's possible only if mask have not cyclic. Default value = false
 `alwaysShowMask`: show mask when input inactive

 `Callbacks`:
   `onChange`(event). event is react-event. If you want access to input value, you may use: `event.target.value`     
   `getReference`: callback to get native input ref

## react-numberinput

Component that allow to format only numbers.

### Installation

```
npm install --save react-numberinput
```

This component work on top of react-maskinput and define custom formatting function called `reformat`. Also you can use this component as example to create you own components based on react-maskinput.

### Usage

```javascript
import NumberInput from 'react-numberinput';

ReactDOM.render(
    someElement, 
    <NumberInput />
)
```

### Props

All of passed props is applying to maskInput directly. See maskInput for more information. Note if you using reformat 
function maskChar, maskString, mask will be ignored.

## mask-input

If you use vanila js, without react, you can install mask-input. This component is similar to react-maskinput, but don't use react.

### Installation

```
npm install --save mask-input
```

### Usage

Mask input receive in constructor props:
```javascript
this.maskInput = new MaskInput(this.refs.maskInput, {
    mask: '0000-0000-0000-0000'
});
```

### How to change params in runtime

To change props you can use setProps method:
```javascript
this.maskInput.setProps({mask: '0000-0000'});
```

VanilaJs maskInput support all props, that support react-maskinput.

## input-core

If you want to use only core, without input decorators:

```
npm install --save input-core
```

# Contributing

1) Fork it!
2) Create your feature branch: `git checkout -b my-new-feature`
3) Commit your changes: `git commit -m 'Add some feature'`
4) Push to the branch: `git push origin my-new-feature`
5) Submit a pull request 

# Changelog

0.1.0 First publish

# TODO

1) Cover all input-core with unit tests
2) Dynamically change props in demo page

# License

MIT

