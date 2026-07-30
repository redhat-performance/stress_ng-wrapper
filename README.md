# stress-ng Benchmark Wrapper

## Description

This wrapper facilitates the automated execution of the stress-ng stress testing tool. stress-ng is a comprehensive Linux stress testing utility that can exercise various subsystems of a computer including CPU, memory, I/O, and OS kernel interfaces. It provides a wide range of stressor methods to thoroughly test system stability and measure performance under load.

The wrapper provides:
- Automated stress-ng execution across 49 predefined stressor configurations.
- Multi-iteration testing with averaged results.
- Result collection, processing, and verification.
- CSV and JSON output formats.
- System configuration metadata capture.
- Integration with test_tools framework.
- Optional Performance Co-Pilot (PCP) integration.

## Command-Line Options

```
stress-ng Wrapper Options:
  --opts_file <path>: Path to the test option file defining which stressors to run.
      Default: test_opts in the script directory.
      Each line in the file has the format: test_name,stress-ng-flags

General test_tools options:
  --home_parent <value>: Parent home directory. If not set, defaults to current working directory.
  --host_config <value>: Host configuration name, defaults to current hostname.
  --iterations <value>: Number of times to run each test, defaults to 1.
  --run_user: User that is actually running the test on the test system. Defaults to current user.
  --sys_type: Type of system working with (aws, azure, hostname). Defaults to hostname.
  --sysname: Name of the system running, used in determining config files. Defaults to hostname.
  --tuned_setting: Used in naming the results directory. For RHEL, defaults to current active tuned profile.
      For non-RHEL systems, defaults to 'none'.
  --use_pcp: Enable Performance Co-Pilot monitoring during test execution.
  --tools_git <value>: Git repo to retrieve the required tools from.
      Default: https://github.com/redhat-performance/test_tools-wrappers
  --usage: Display this usage message.
```

## What the Script Does

The `stress_ng_run` script performs the following workflow:

1. **Environment Setup**:
   - Clones the test_tools-wrappers repository if not present (default: ~/test_tools).
   - Tries wget, then curl, then git clone to obtain the tools.
   - Sources error codes and general setup utilities.
   - Gathers system hardware information.

2. **Package Installation**:
   - Installs required dependencies via package_tool: bc, stress-ng.
   - Dependencies are defined in stress_ng.json for different OS variants.

3. **PCP Setup** (optional):
   - If `--use_pcp` is specified, initializes Performance Co-Pilot monitoring.
   - Creates a timestamped PCP data directory at `/tmp/pcp_<timestamp>/`.
   - Resets all OpenMetrics values between iterations.

4. **Test Execution**:
   - Reads the test configuration file (`test_opts` by default) line by line.
   - Each line defines a test name and corresponding stress-ng flags.
   - All stressors run with `-1` (one instance per CPU core) and `--no-rand-seed` for reproducibility.
   - Runs each stressor for the configured number of iterations (`--iterations`).
   - Records start and end timestamps for each test run.
   - Saves raw output to `results_<test_name>_iteration_<N>.data`.

5. **Data Collection**:
   - Extracts bogo ops/sec from each iteration's raw output.
   - Averages values across iterations using `bc`.
   - Generates `results_stress_ng.csv` with configuration and performance data.
   - Optionally records PCP performance data during execution.

6. **Verification**:
   - Converts CSV to JSON via `csv_to_json`.
   - Validates results against Pydantic schema (`result_schema.py`) ensuring:
     - All test names match expected Benchmark enum values.
     - All bogo_ops_per_sec values are positive integers.
     - Timestamps are valid datetime objects.

7. **Output**:
   - Creates timestamped results directory in `${HOME}/export_results/stress_ng_<YYYY.MM.DD-HH.MM.SS>`.
   - Saves all raw output files, processed CSV/JSON, and system metadata.
   - Optionally saves PCP performance data.
   - Archives results to configured storage location.

## Dependencies

Location of underlying workload: https://github.com/ColinIanKing/stress-ng

**General packages required**: bc, stress-ng

To run:
```bash
git clone https://github.com/redhat-performance/stress_ng-wrapper
cd stress_ng-wrapper/stress_ng
./stress_ng_run
```

The script will automatically detect your CPU configuration and run all stressors with one instance per core.

## The stress-ng Stressors

The default `test_opts` file defines 49 stressor configurations organized into the following categories:

### CPU and Math

1. **hash**: Cryptographic hashing stressor testing hash computation throughput.
2. **cpu_stress**: CPU stressor running all available CPU methods (`--cpu-method all`).
3. **power_math**: Power/exponentiation math operations.
4. **matrix_math**: Matrix arithmetic operations (120s duration).
5. **vector_math**: Wide vector math operations.
6. **integer_math**: Integer math operations.
7. **floating_point**: Floating-point arithmetic operations.
8. **matrix_3d_math**: 3D matrix arithmetic operations.
9. **exponential_math**: Exponential math functions.
10. **logarithmic_math**: Logarithmic math functions.
11. **trigonometric_math**: Trigonometric math functions.
12. **hyperbolic_trigonometric_math**: Hyperbolic trigonometric functions.
13. **fused_multiply_add**: FMA instruction workload.
14. **bessel_math_operations**: Bessel function math operations.
15. **integer_bit_operations**: Integer bitwise operations.
16. **function_call**: Function call overhead stressor.
17. **fractal_Generator**: Fractal generation workload.
18. **avx-512_VNNI**: AVX-512 VNNI instruction stressor.
19. **vector_floating_point**: Vector floating-point operations.
20. **vector_shuffle**: Vector shuffle operations.
21. **wide_Vector_Math**: Wide vector math operations.
22. **jpeg_compression**: JPEG compression workload.

### Memory

23. **mmap**: Memory-mapped file stressor.
24. **malloc**: Memory allocation stressor (180s duration).
25. **memfd**: Memory file descriptor stressor.
26. **memory_copying**: Memory copy operations (180s duration).
27. **cpu_cache**: CPU cache stressor.

### IPC and Synchronization

28. **pipe**: Pipe I/O stressor.
29. **poll**: Poll system call stressor.
30. **futex**: Futex (fast userspace mutex) stressor.
31. **mutex**: Mutex lock/unlock stressor.
32. **semaphores**: Semaphore operations stressor.
33. **socket_activity**: Socket I/O stressor with zero-copy option.
34. **system_v_message_passing**: System V message queue stressor.

### Process and Thread

35. **cloning**: Process clone stressor.
36. **forking**: Process fork stressor.
37. **pthread**: POSIX thread creation stressor.
38. **context_switching**: Context switch stressor.
39. **mixed_scheduler**: Mixed scheduler workload.

### Data Structures and Sorting

40. **avl_tree**: AVL tree operations (`--tree-method avl`).
41. **radix_String_sort**: Radix string sort stressor.
42. **bitonic_integer_sort**: Bitonic integer sort stressor.
43. **glibc_qsort_data_sorting**: glibc qsort sorting stressor.

### String and Regex

44. **glibc_c_string_functions**: glibc C string function stressor.
45. **posix_regular_expressions**: POSIX regex stressor.

### I/O

46. **sendfile**: sendfile system call stressor.

### Compression and Crypto

47. **zlib**: zlib compression/decompression stressor.
48. **crypto**: Cryptographic operations stressor.

### NUMA

49. **numa**: NUMA memory policy stressor.

### Performance Metrics

Each stressor reports **bogo ops/sec** (bogus operations per second) -- a throughput metric indicating how many operations the stressor completed per second. Higher values indicate better performance. When running multiple iterations, bogo ops/sec values are averaged across all iterations.

## Test Configuration File Format

The `test_opts` file uses a simple CSV format with one stressor per line:

```
test_name,stress-ng-flags
```

For example:
```
hash,--hash -1 --no-rand-seed -t 30
malloc,--malloc -1 --no-rand-seed -t 180
matrix_math,--matrix -1 --no-rand-seed -t 120
```

Common flags used across all tests:
- `-1`: Run one stressor instance per CPU core.
- `--no-rand-seed`: Disable random seeding for reproducible results.
- `-t <seconds>`: Test duration (30s for most tests; 120s for matrix_math; 180s for malloc and memory_copying).

You can create a custom test configuration file and pass it with `--opts_file`.

## Output Files

The results directory contains:

- **results_stress_ng.csv**: CSV file with all stressor bogo ops/sec metrics
- **results_stress_ng.json**: JSON file with validated results data
- **results_\<test\>_iteration_\<N\>.data**: Raw stress-ng output per stressor per iteration
- **test_results_report**: Detailed test execution report
- **meta_data\*.yml**: System metadata (CPU info, memory, kernel version)
- **PCP data** (if --use_pcp option used): Performance Co-Pilot monitoring data

## Examples

### Basic run with defaults
```bash
./stress_ng_run
```
This runs with:
- All 49 stressors from the default test_opts file
- One instance per CPU core for each stressor
- 1 iteration of each stressor
- Default durations (30s for most, 120s/180s for heavier tests)

### Run multiple iterations
```bash
./stress_ng_run --iterations 3
```
Runs each stressor 3 times and averages the bogo ops/sec results for consistency validation.

### Run with custom test configuration
```bash
./stress_ng_run --opts_file /path/to/custom_opts
```
Uses a custom test configuration file instead of the default `test_opts`.

### Run with PCP monitoring
```bash
./stress_ng_run --use_pcp
```
Collects Performance Co-Pilot data during the run for detailed performance analysis.

### Combination example
```bash
./stress_ng_run --iterations 5 --use_pcp --opts_file my_tests
```
Runs 5 iterations with PCP monitoring using a custom test configuration.

## How Result Averaging Works

When running multiple iterations (--iterations > 1):

1. Each iteration produces raw output for every stressor.
2. Raw results are saved in separate `results_<test>_iteration_<N>.data` files.
3. The wrapper extracts bogo ops/sec from the last metric line in each file.
4. Final results are the arithmetic mean across all iterations, computed with `bc`.
5. CSV output contains the averaged values along with start and end timestamps.

This approach:
- Reduces impact of transient system effects
- Provides more reliable performance measurements
- Helps identify result variance across runs

## Return Codes

The script uses standardized error codes from test_tools error_codes:
- **0**: Success
- **101**: Git clone failure (unable to retrieve test_tools)
- **E_GENERAL**: General execution errors (package installation failures, test execution failures, validation failures)
- **E_USAGE**: Invalid usage/arguments

Exit codes indicate specific failure points for automated testing workflows.

## Notes

### Supported Platforms
- **Linux**: x86_64 and aarch64 architectures
- **OS Support**: RHEL, Ubuntu, SLES, Amazon Linux

### Test Duration
Total runtime depends on the test configuration:
- Default `test_opts` with 49 stressors: approximately 30-40 minutes per iteration
  - Most stressors run for 30 seconds each
  - `matrix_math` runs for 120 seconds
  - `malloc` and `memory_copying` run for 180 seconds each
- Custom configurations with fewer stressors will complete faster

### Performance Considerations
- stress-ng stressors exercise different system subsystems:
  - **CPU-bound**: cpu_stress, integer_math, floating_point, matrix_math, trigonometric_math
  - **Memory-bound**: mmap, malloc, memory_copying, cpu_cache
  - **IPC/Kernel**: pipe, poll, futex, mutex, semaphores, socket_activity
  - **Process/Thread**: cloning, forking, pthread, context_switching
- Run multiple iterations (--iterations 3 or higher) for reliable results
- Ensure system is idle during testing for best consistency

### Performance Tips
- Run multiple iterations (--iterations 3+) to verify consistency
- Ensure system is idle (no other workloads) for best results
- Disable CPU frequency scaling (use performance governor) for reproducible results
- Consider the active tuned profile on RHEL systems
- For production benchmarking, allow system to warm up with a test run first
- PCP monitoring (--use_pcp) adds minimal overhead but provides detailed metrics
- Create a focused `test_opts` file with fewer stressors for quick targeted testing

### Custom Test Configuration
To create a custom test configuration:
1. Copy `test_opts` to a new file.
2. Add, remove, or modify stressor lines.
3. Each line must follow the format: `test_name,stress-ng-flags`.
4. Ensure test names match entries in `result_schema.py` if result validation is required.
5. Pass the custom file with `--opts_file`.

### Troubleshooting
- If stress-ng is not found, verify it is installed (`dnf install stress-ng` or equivalent)
- If results seem inconsistent, run more iterations and check system load during testing
- Use `--use_pcp` to collect detailed performance counters for analysis
- Check `results_<test>_iteration_<N>.data` files for detailed error messages
- If the NUMA stressor fails, verify NUMA is available on your system
- If AVX-512 VNNI stressor fails, verify your CPU supports VNNI instructions
- For upstream stress-ng issues, see: https://github.com/ColinIanKing/stress-ng

## References

- stress-ng GitHub: https://github.com/ColinIanKing/stress-ng
- stress-ng Documentation: https://github.com/ColinIanKing/stress-ng/blob/master/README.md
- stress-ng man page: https://manpages.ubuntu.com/manpages/noble/man1/stress-ng.1.html
- test_tools Framework: https://github.com/redhat-performance/test_tools-wrappers
