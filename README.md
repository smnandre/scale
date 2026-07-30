# Alto \ Scale

A strict, mathematical engine for **Modular Scales**.
It handles the arithmetic of progressions, designed for generative design, typography engines, and rhythmic computation.

---

&nbsp; [![PHP Version](https://img.shields.io/badge/PHP-8.4+-ffefdf?logoColor=white&labelColor=000)](https://github.com/altophp/scale)
&nbsp; [![Packagist Version](https://img.shields.io/packagist/v/alto/scale?label=Stable&logoColor=white&logoSize=auto&labelColor=000&color=ffefdf)](https://packagist.org/packages/alto/scale)
&nbsp; [![CI](https://img.shields.io/github/actions/workflow/status/altophp/scale/CI.yml?branch=main&label=Tests&logoColor=white&logoSize=auto&labelColor=000&color=ffefdf)](https://github.com/altophp/scale/actions)
&nbsp; [![PHP Version](https://img.shields.io/badge/PHPUnit-100%25-ffefdf?logoColor=white&labelColor=000)](https://github.com/altophp/scale)
&nbsp; [![PHP Version](https://img.shields.io/badge/PHPStan-LVL%2010-ffefdf?logoColor=white&labelColor=000)](https://github.com/altophp/scale)
&nbsp; [![License](https://img.shields.io/github/license/altophp/scale?label=License&logoColor=white&logoSize=auto&labelColor=000&color=ffefdf)](./LICENSE)


## Installation

```bash
composer require alto/scale
```

## Core Concepts

### 1. The Scale Facade

Use the `Scale` facade for fluent instantiation of any scale type.

```php
use Alto\Scale\Scale;
use Alto\Scale\Ratio;

// Modular (Geometric): Base 16px, Ratio 1.25 (Major Third)
$type = Scale::majorThird(16.0);

// Linear (Arithmetic): Base 0, Increment 8
$grid = Scale::linear(0, 8);

// Fibonacci: Using Binet's formula
$fib = Scale::fibonacci();

// Multi-Strand: Interleaved scales (e.g., 16px and 12px strands)
$strands = Scale::strands([16, 12], Ratio::PerfectFifth);
```

### 2. Computing Values

All scales implement the `ScaleInterface` and are `IteratorAggregate`.

```php
echo $type->get(0);  // 16.0
echo $type->get(1);  // 20.0
echo $type->get(-1); // 12.8

// ModularScale is callable
echo $type(2); // 25.0
```

#### Snapping (Normalization)

Find the nearest mathematical step for any arbitrary value.

```php
// Returns 20.0 (The nearest step on a Major Third scale starting at 16)
$value = $type->snap(19.8);
```

#### Iteration & Ranges

```php
// Scales are iterable (defaults to steps 0-10)
foreach ($type as $step => $value) {
    // ...
}

// Custom ranges
$values = $type->range(-2, 2);
// Returns: [-2 => 10.24, -1 => 12.8, 0 => 16.0, 1 => 20.0, 2 => 25.0]
```

### 3. The Guesser (Reverse Engineering)

Analyze messy datasets to find their underlying mathematical scale.

```php
use Alto\Scale\Scale;
use Alto\Scale\ScaleGuesser;

$messyValues = [15.9, 20.1, 24.8, 31.5];

// Via the facade
$scale = Scale::guess($messyValues);

// Or directly with custom tolerance
$guesser = new ScaleGuesser(tolerance: 0.05);
$scale = $guesser->guess($messyValues);

// Align values to the guessed (or provided) scale
$aligned = $guesser->align($messyValues);
```

### 4. The Linter (Audit & Fix)

Audit datasets for scale compliance and automatically harmonize them.

```php
use Alto\Scale\ScaleLinter;
use Alto\Scale\Scale;

$linter = new ScaleLinter(Scale::linear(0, 8));

// Get a detailed compliance report
$report = $linter->lint([8, 15, 24]);
// Each entry: ['original', 'suggested', 'step', 'deviation', 'isValid']

// Returns [8.0, 16.0, 24.0]
$fixed = $linter->fix([8, 15, 24]);
```

## API Reference

### `Alto\Scale\Scale` (Facade)

| Method                                       | Returns            |
|----------------------------------------------|--------------------|
| `modular(float $base, float\|Ratio $ratio)`  | `ModularScale`     |
| `linear(float $base, float $increment)`      | `LinearScale`      |
| `fibonacci(float $multiplier)`               | `FibonacciScale`   |
| `strands(array $bases, float\|Ratio $ratio)` | `MultiStrandScale` |
| `guess(array $values)`                       | `ModularScale`     |
| `majorThird(float $base)`                    | `ModularScale`     |
| `perfectFifth(float $base)`                  | `ModularScale`     |
| `golden(float $base)`                        | `ModularScale`     |

### `Alto\Scale\ScaleInterface`

All scale classes (`ModularScale`, `LinearScale`, `FibonacciScale`, `MultiStrandScale`) implement:

| Method                             | Description                             |
|------------------------------------|-----------------------------------------|
| `get(int $step): float`            | Get the value at a specific step        |
| `stepOf(float $value): int`        | Find the closest step index for a value |
| `snap(float $value): float`        | Snap a value to the nearest step        |
| `range(int $min, int $max): array` | Generate values between steps           |

### `Alto\Scale\ModularScale`

Geometric progression: `value = base * (ratio ^ step)`

| Method                                                  | Description                                  |
|---------------------------------------------------------|----------------------------------------------|
| `areHarmonic(float $a, float $b, float $epsilon): bool` | Check if two values are harmonically related |
| `withBase(float $base): self`                           | Create a new scale with a different base     |
| `withRatio(float\|Ratio $ratio): self`                  | Create a new scale with a different ratio    |
| `shift(int $steps): self`                               | Create a new scale shifted by N steps        |

### `Alto\Scale\LinearScale`

Arithmetic progression: `value = base + (step * increment)`

### `Alto\Scale\FibonacciScale`

Fibonacci sequence using Binet's formula, with optional multiplier.

### `Alto\Scale\MultiStrandScale`

Interleaved modular scales sharing a common ratio, useful for multi-strand typographic scales.

### `Alto\Scale\ScaleGuesser`

| Method                                                | Description                       |
|-------------------------------------------------------|-----------------------------------|
| `guess(array $values): ModularScale`                  | Infer a modular scale from values |
| `align(array $values, ?ScaleInterface $scale): array` | Snap all values to a scale        |

### `Alto\Scale\ScaleLinter`

| Method                       | Description                   |
|------------------------------|-------------------------------|
| `lint(array $values): array` | Audit values against a scale  |
| `fix(array $values): array`  | Harmonize values to the scale |

### `Alto\Scale\Ratio` (Enum)

Standard musical/typographic ratios:

| Ratio             | Value |
|-------------------|-------|
| `MinorSecond`     | 1.067 |
| `MajorSecond`     | 1.125 |
| `MinorThird`      | 1.200 |
| `MajorThird`      | 1.250 |
| `PerfectFourth`   | 1.333 |
| `AugmentedFourth` | 1.414 |
| `PerfectFifth`    | 1.500 |
| `MinorSixth`      | 1.600 |
| `GoldenRatio`     | 1.618 |
| `MajorSixth`      | 1.667 |
| `MinorSeventh`    | 1.778 |
| `MajorSeventh`    | 1.875 |
| `Octave`          | 2.000 |

## License

This project is licensed under the [MIT License](./LICENSE).
