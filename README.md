# Statistical Moments and Validation Utilities for PyEyesWeb

A lightweight Python component for computing **statistical moments from real-time multivariate signal windows**, together with reusable validation helpers used across the PyEyesWeb codebase.

The module focuses on four core distribution descriptors:

- mean
- standard deviation
- skewness
- kurtosis

It is designed for streaming sensor data, motion analysis, feature extraction, anomaly detection, and signal-quality assessment.

---

## Overview

Statistical moments provide a compact description of the shape and behaviour of a signal distribution.

For a multivariate sliding window, this module computes each requested statistic independently for every feature column.

```text
Sliding-window signal data
          ↓
Window completeness check
          ↓
Requested moment selection
          ↓
Column-wise computation
          ↓
Dictionary of scalar or vector results
```

The same repository also includes reusable validators for:

- numeric parameters
- integer parameters
- boolean flags
- lists and option sets
- bounded ranges
- filter parameter tuples
- window sizes

---

## Main Components

```text
pyeyesweb/
├── analysis/
│   └── statistical_moment.py
├── utils/
│   └── validators.py
└── data_models/
    └── sliding_window.py
```

Suggested filenames:

```text
StatisticalMoment.py
validators.py
```

---

## Features

- real-time statistical analysis from `SlidingWindow`
- support for univariate and multivariate signals
- selective computation of requested moments
- sample standard deviation with `ddof=1`
- SciPy-based skewness and kurtosis
- automatic scalar output for one feature
- list output for multiple features
- callable analyzer interface
- reusable parameter-validation utilities
- consistent error messages across modules
- filter-parameter normalization support

---

## Requirements

- Python 3.9 or later
- NumPy
- SciPy
- PyEyesWeb

Install the external dependencies with:

```bash
pip install numpy scipy
```

The analyzer imports:

```python
from pyeyesweb.data_models.sliding_window import SlidingWindow
```

The filter validator optionally imports:

```python
from pyeyesweb.utils.signal_processing import validate_filter_params
```

Make sure those PyEyesWeb modules are available on the Python path.

---

## Quick Start

```python
from pyeyesweb.analysis.statistical_moment import StatisticalMoment

analyzer = StatisticalMoment()

result = analyzer(
    sliding_window,
    methods=[
        "mean",
        "std_dev",
        "skewness",
        "kurtosis",
    ],
)

print(result)
```

Example output for a three-feature signal:

```python
{
    "mean": [0.15, -0.03, 1.42],
    "std": [0.81, 0.54, 0.22],
    "skewness": [0.10, -0.41, 0.07],
    "kurtosis": [-0.72, 0.18, 1.24],
}
```

---

## Supported Methods

| Input method name | Output key | Computation |
|---|---|---|
| `"mean"` | `"mean"` | Arithmetic mean |
| `"std_dev"` | `"std"` | Sample standard deviation |
| `"skewness"` | `"skewness"` | Distribution asymmetry |
| `"kurtosis"` | `"kurtosis"` | Fisher excess kurtosis |

Only requested methods are computed.

Unknown method names are currently ignored silently.

---

## Basic Usage with a Sliding Window

```python
from pyeyesweb.analysis.statistical_moment import StatisticalMoment
from pyeyesweb.data_models.sliding_window import SlidingWindow

window = SlidingWindow(
    max_length=50,
    n_columns=3,
)

for sample in signal_stream:
    window.append(sample)

analyzer = StatisticalMoment()

result = analyzer.compute_statistics(
    window,
    methods=["mean", "std_dev"],
)

print(result)
```

When the window is not full, the method returns:

```python
np.nan
```

---

## Univariate Output

For a one-column signal:

```python
result = analyzer(
    window,
    methods=["mean", "std_dev"],
)
```

The result contains scalar values:

```python
{
    "mean": 0.42,
    "std": 0.18,
}
```

---

## Multivariate Output

For a signal with multiple columns:

```python
{
    "mean": [0.42, 0.81, -0.12],
    "std": [0.18, 0.21, 0.09],
}
```

The order of returned values follows the input feature-column order.

---

## Statistical Definitions

### Mean

The arithmetic mean measures central tendency:

```text
μ = (1 / n) Σ xᵢ
```

Implementation:

```python
np.mean(data, axis=0)
```

### Standard deviation

The module uses the sample standard deviation:

```text
s = √[Σ(xᵢ − x̄)² / (n − 1)]
```

Implementation:

```python
np.std(data, axis=0, ddof=1)
```

### Skewness

Skewness measures distribution asymmetry.

Interpretation:

- near `0`: approximately symmetric
- positive: longer right tail
- negative: longer left tail

Implementation:

```python
scipy.stats.skew(data, axis=0)
```

### Kurtosis

Kurtosis measures tail heaviness and concentration.

SciPy returns Fisher excess kurtosis by default:

- near `0`: similar to a normal distribution
- positive: heavier tails
- negative: lighter tails

Implementation:

```python
scipy.stats.kurtosis(data, axis=0)
```

---

## API Reference

## `StatisticalMoment`

```python
StatisticalMoment()
```

The class currently has no constructor parameters.

---

### `compute_statistics(signals, methods)`

Computes selected statistical moments from a full sliding window.

```python
result = analyzer.compute_statistics(
    signals,
    methods=["mean", "std_dev"],
)
```

Parameters:

| Parameter | Type | Description |
|---|---|---|
| `signals` | `SlidingWindow` | Full multivariate signal window |
| `methods` | `list[str]` | Requested statistics |

Returns:

- `dict` when computation succeeds
- `np.nan` when the window is not full
- `np.nan` when fewer than two samples are available

---

### `__call__(sliding_window, methods)`

Callable wrapper around `compute_statistics`.

```python
result = analyzer(
    sliding_window,
    ["mean", "skewness"],
)
```

---

## Validation Utilities

The validation module provides consistent parameter checks for PyEyesWeb components.

---

### `validate_numeric`

```python
validate_numeric(
    value,
    name,
    min_val=None,
    max_val=None,
)
```

Validates an integer or floating-point parameter and returns it as `float`.

Example:

```python
rate = validate_numeric(
    50,
    "rate_hz",
    min_val=0.1,
    max_val=100000,
)
```

Raises:

- `TypeError` for non-numeric values
- `ValueError` for values outside the configured bounds

---

### `validate_integer`

```python
validate_integer(
    value,
    name,
    min_val=None,
    max_val=None,
)
```

Example:

```python
window_size = validate_integer(
    100,
    "window_size",
    min_val=1,
    max_val=10000,
)
```

---

### `validate_boolean`

```python
enabled = validate_boolean(
    True,
    "output_interpretation",
)
```

Unlike Python truthiness checks, this function rejects integers such as `0` and `1`.

---

### `validate_list`

```python
methods = validate_list(
    ["mean", "std_dev"],
    "methods",
    valid_options=[
        "mean",
        "std_dev",
        "skewness",
        "kurtosis",
    ],
    min_length=1,
)
```

It can validate:

- input type
- minimum length
- maximum length
- allowed element values

---

### `validate_range`

```python
threshold = validate_range(
    0.7,
    "threshold",
    0.0,
    1.0,
)
```

This helper assumes the input already supports comparison.

---

### `validate_filter_params_tuple`

```python
params = validate_filter_params_tuple(
    (1.0, 10.0, 100.0),
)
```

Expected structure:

```text
(lowcut, highcut, sampling_frequency)
```

The function accepts a tuple or list and returns a tuple.

---

### `validate_and_normalize_filter_params`

```python
params = validate_and_normalize_filter_params(
    (1.0, 10.0, 100.0),
)
```

This function:

1. allows `None`
2. validates tuple structure
3. calls the shared signal-processing validator
4. returns normalized filter parameters

---

### `validate_window_size`

```python
window_size = validate_window_size(250)
```

Equivalent to:

```python
validate_integer(
    value,
    "window_size",
    min_val=1,
    max_val=10000,
)
```

---

## Example: Motion Feature Extraction

```python
analyzer = StatisticalMoment()

features = analyzer(
    movement_window,
    methods=[
        "mean",
        "std_dev",
        "skewness",
        "kurtosis",
    ],
)

if isinstance(features, dict):
    movement_feature_vector = [
        *features["mean"],
        *features["std"],
        *features["skewness"],
        *features["kurtosis"],
    ]
```

This can support:

- movement classification
- anomaly detection
- activity recognition
- gait analysis
- behavioural signal modelling

---

## Example: Selecting Only Required Features

```python
result = analyzer(
    window,
    methods=["mean", "std_dev"],
)
```

This avoids unnecessary skewness and kurtosis calculations when only first- and second-order statistics are needed.

---

## Example: Parameter Validation in Another Module

```python
from pyeyesweb.utils.validators import (
    validate_boolean,
    validate_integer,
    validate_list,
    validate_numeric,
)

class SignalAnalyzer:
    def __init__(
        self,
        window_size=100,
        threshold=0.5,
        enabled=True,
        methods=None,
    ):
        if methods is None:
            methods = ["mean", "std_dev"]

        self.window_size = validate_integer(
            window_size,
            "window_size",
            min_val=1,
            max_val=10000,
        )

        self.threshold = validate_numeric(
            threshold,
            "threshold",
            min_val=0.0,
            max_val=1.0,
        )

        self.enabled = validate_boolean(
            enabled,
            "enabled",
        )

        self.methods = validate_list(
            methods,
            "methods",
            valid_options=[
                "mean",
                "std_dev",
                "skewness",
                "kurtosis",
            ],
            min_length=1,
        )
```

---

## Recommended Project Structure

```text
pyeyesweb/
├── analysis/
│   ├── __init__.py
│   └── statistical_moment.py
├── data_models/
│   ├── __init__.py
│   └── sliding_window.py
├── utils/
│   ├── __init__.py
│   ├── validators.py
│   └── signal_processing.py
└── tests/
    ├── test_statistical_moment.py
    └── test_validators.py
```

---

## Testing

Run all tests:

```bash
pytest -v
```

Run only statistical-moment tests:

```bash
pytest tests/test_statistical_moment.py -v
```

Run only validator tests:

```bash
pytest tests/test_validators.py -v
```

---

## Suggested Tests for `StatisticalMoment`

The test suite should cover:

- incomplete window
- one-feature input
- multi-feature input
- mean computation
- sample standard deviation
- skewness computation
- kurtosis computation
- selective method execution
- unknown method handling
- fewer than two samples
- constant-valued features
- NaN and infinite inputs
- callable interface

Example:

```python
def test_multivariate_mean(full_window):
    analyzer = StatisticalMoment()

    result = analyzer(
        full_window,
        methods=["mean"],
    )

    assert "mean" in result
    assert isinstance(result["mean"], list)
```

---

## Suggested Tests for Validators

The validation test suite should cover:

- valid numeric input
- numeric lower and upper bounds
- rejection of strings
- integer validation
- rejection of booleans as integers where relevant
- boolean validation
- list length constraints
- invalid list options
- range bounds
- malformed filter tuples
- non-numeric filter elements
- `None` filter parameters
- valid window sizes

---

## Important Behaviour Notes

### Inconsistent return type

`compute_statistics` returns either:

```python
dict
```

or:

```python
np.nan
```

This makes type handling less predictable.

A more consistent design would return:

```python
{
    "ready": False,
    "statistics": None,
}
```

or an empty dictionary.

### Invalid methods are ignored

Unknown method names are skipped silently:

```python
methods=["mean", "invalid"]
```

returns only the mean.

For stricter behaviour, validate methods before computation:

```python
validate_list(
    methods,
    "methods",
    valid_options=[
        "mean",
        "std_dev",
        "skewness",
        "kurtosis",
    ],
)
```

### Output key mismatch

The input method name is:

```text
std_dev
```

but the output key is:

```text
std
```

This should be documented or standardized.

### Constant features

Skewness and kurtosis may return `NaN` for constant-valued signals because the variance is zero.

### Missing-value handling

The implementation uses standard NumPy and SciPy functions, not their NaN-aware alternatives.

Inputs containing `NaN` can propagate `NaN` into the results.

### Bias correction

SciPy's default skewness and kurtosis settings may include bias in finite samples.

Research applications may prefer:

```python
stats.skew(data, axis=0, bias=False)
stats.kurtosis(data, axis=0, bias=False)
```

### Fisher kurtosis

The current implementation uses Fisher kurtosis, where a normal distribution has expected kurtosis near zero.

### Boolean values in numeric validation

In Python, `bool` is a subclass of `int`.

The current `validate_numeric` accepts:

```python
True
False
```

as numeric values.

The current `validate_integer` also accepts booleans because:

```python
isinstance(True, int)
```

is `True`.

If this is undesirable, explicitly reject booleans.

---

## Recommended Improvements

### Validate the method list

```python
VALID_METHODS = [
    "mean",
    "std_dev",
    "skewness",
    "kurtosis",
]

methods = validate_list(
    methods,
    "methods",
    valid_options=VALID_METHODS,
    min_length=1,
)
```

### Return a consistent result object

```python
{
    "ready": True,
    "sample_size": n_samples,
    "feature_dimension": n_features,
    "statistics": result,
}
```

### Add NaN policies

Support options such as:

```python
nan_policy="propagate"
nan_policy="omit"
nan_policy="raise"
```

### Add bias configuration

```python
StatisticalMoment(
    bias=False,
    fisher=True,
)
```

### Add feature names

Instead of position-only lists:

```python
{
    "mean": {
        "x": 0.15,
        "y": 0.22,
        "z": 0.91,
    }
}
```

### Use structured type hints

```python
from typing import Literal, Sequence, TypedDict
```

### Improve validation consistency

Reject booleans explicitly in numeric and integer validators:

```python
if isinstance(value, bool):
    raise TypeError(...)
```

---

## Performance

For a window with:

- `n` samples
- `d` features

the basic statistical calculations are approximately:

```text
O(n × d)
```

Memory usage is dominated by the sliding-window array.

The module is suitable for small and medium streaming windows. For very high-frequency signals, consider incremental or online moment algorithms that avoid recalculating statistics over the entire window.

---

## Online Alternatives

For high-throughput streaming applications, future versions could use:

- Welford's online mean and variance
- online skewness and kurtosis algorithms
- rolling NumPy or pandas operations
- Numba acceleration
- vectorized batch updates

These approaches may reduce repeated computation for overlapping windows.

---

## Research Considerations

Statistical moments are useful but should not be interpreted in isolation.

For signal characterization, consider combining them with:

- median
- interquartile range
- entropy
- RMS
- zero-crossing rate
- spectral energy
- dominant frequency
- autocorrelation
- Hjorth parameters
- wavelet features
- clusterability measures

---

## References

- Pearson, K. (1895). *Contributions to the Mathematical Theory of Evolution*.
- Fisher, R. A. (1925). *Statistical Methods for Research Workers*.
- SciPy documentation for `scipy.stats.skew`.
- SciPy documentation for `scipy.stats.kurtosis`.

---

## Contributing

Contributions are welcome for:

- stricter validation
- richer statistical descriptors
- online algorithms
- improved type hints
- NaN handling
- test coverage
- performance benchmarking
- documentation improvements

Suggested workflow:

```bash
git checkout -b feature/improve-statistical-moments
pytest -v
git add .
git commit -m "Improve statistical moments analysis"
git push origin feature/improve-statistical-moments
```

## Disclaimer

This module provides general-purpose signal statistics.

It does not independently provide clinical, behavioural, or diagnostic conclusions. Any use in healthcare, psychology, or human-movement research requires suitable validation, study design, and domain interpretation.
