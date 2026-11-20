# Salon receipt rounding examples

A small executable JavaScript example for converting a percentage into payable cents. It uses integer cents and basis points, so a half-cent has an explicit rounding rule instead of depending on binary floating-point dollars.

This is an educational example, separate from the Calculator Toolkit application source. No dependencies or account are needed to run it locally. Copy the following block into `receipt.cjs` and run `node receipt.cjs` with a current Node.js release supporting BigInt.

## Calculation contract

- The base is a non-negative safe integer in cents.
- The percentage is a non-negative integer in basis points, up to 10,000 (100%).
- Round the final gratuity half up to the nearest cent, once.
- Keep the receipt total and tip base separate: retail and tax can affect payment without changing the selected service-only tip base.
- This example does not decide etiquette, mandatory charges, or how workers share tips.

```js
const assert = require('node:assert/strict');

function tipCents(baseCents, basisPoints) {
  if (!Number.isSafeInteger(baseCents) || baseCents < 0) {
    throw new RangeError('baseCents must be a non-negative safe integer');
  }
  if (!Number.isSafeInteger(basisPoints) || basisPoints < 0 || basisPoints > 10000) {
    throw new RangeError('basisPoints must be an integer from 0 to 10000');
  }
  const numerator = BigInt(baseCents) * BigInt(basisPoints);
  return Number((numerator + 5000n) / 10000n);
}

// $42.50 at 15% is $6.375, rounded once to $6.38.
assert.equal(tipCents(4250, 1500), 638);
assert.equal(tipCents(1000, 1800), 180);
assert.equal(tipCents(4250, 0), 0);
assert.equal(tipCents(0, 2000), 0);
assert.equal(tipCents(Number.MAX_SAFE_INTEGER, 10000), Number.MAX_SAFE_INTEGER);
assert.throws(() => tipCents(-1, 2000), RangeError);
assert.throws(() => tipCents(10.5, 2000), RangeError);
assert.throws(() => tipCents(1000, 10001), RangeError);

// $90 service + $15 retail + $6.50 tax; an optional 18% service-only tip.
const service = 9000;
const retail = 1500;
const tax = 650;
const tip = tipCents(service, 1800);
assert.equal(tip, 1620);
assert.equal(service + retail + tax + tip, 12770);
// Changing retail does not change the selected tip base.
assert.equal(service + (retail + 700) + tax + tip, 13470);

console.log('All 11 receipt assertions passed.');
```

## Why the denominator is 10,000

One basis point is 0.01 percentage point. Multiplying cents by basis points produces ten-thousandths of a cent. For non-negative values, adding 5,000 before integer division by 10,000 implements round-half-up. The output remains at most the input base because this example caps the rate at 100%.

Production input parsing, refunds, currency conversion, accounting rules and payment processing are outside this example. Validate dollar text before converting it to cents; do not assume `parseFloat(text) * 100` always gives an integer.

## Receipt context

The [Calculator Toolkit hairdresser tipping guide](https://calculator-toolkit.com/how-much-to-tip-hairdresser/) provides salon receipt examples and explains questions to ask about service charges and assistants. Use it as context for choosing test inputs, rather than as an arithmetic dependency.

Disclosure: this repository is published by the Calculator Toolkit owner. The example and explanatory text were prepared with AI assistance and checked by executing the assertions. It is not the application's source code and makes no claim about endorsement by GitHub or another publisher.