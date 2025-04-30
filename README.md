
# LaSR Experiments Setup

This repository contains scripts and setup instructions for running **LaSR (Library-Augmented Symbolic Regression)**, a symbolic regression framework that integrates large language models (LLMs) with evolutionary search to discover interpretable mathematical expressions from data.

## Overview

This project explores LaSR's performance on datasets of different dimensionalities using synthetic equations.

---

## Setup Instructions (Google Colab / Linux)

### 1. Install Julia (v1.9.3)
```bash
!wget https://julialang-s3.julialang.org/bin/linux/x64/1.9/julia-1.9.3-linux-x86_64.tar.gz
!tar -xvzf julia-1.9.3-linux-x86_64.tar.gz
!sudo mv julia-1.9.3 /opt/julia
!sudo ln -s /opt/julia/bin/julia /usr/local/bin/julia
```

### 2. Verify Installation
```bash
!julia --version
!jupyter kernelspec list
```

### 3. Install Required Julia Packages
```bash
!julia -e 'using Pkg; Pkg.add("LibraryAugmentedSymbolicRegression")'
!julia -e 'using Pkg; Pkg.add("MLJ")'
!julia -e 'using Pkg; Pkg.status()'
!julia -e 'using LibraryAugmentedSymbolicRegression; println("Package loaded successfully!")'
```

---

## Example Usage (in `script.jl`)

### 5D Example
```julia
X = (
    a = rand(500),
    b = rand(500),
    c = rand(500),
    d = rand(500),
    e = rand(500),
)

y = @. 2 * cos(X.a * 23.5) - X.b^2 + 0.5 * X.c - 0.3 * X.d^2 + sin(X.e * 3)
y = y .+ randn(500) .* 1e-3  # Add small noise
```

### Initialize LaSR Regressor
```julia
model = LaSRRegressor(
    niterations=40,
    binary_operators=[+, -, *, /, ^],
    unary_operators=[cos],
    populations=20,
    use_llm=true,
    use_concepts=true,
    use_concept_evolution=true,
    llm_operation_weights=LLMOperationWeights(llm_crossover=0.001, llm_mutate=0.001, llm_randomize=0.001),
    llm_context="We believe the relationship between the theta and offset parameter is a function of the cosine of the theta variable and the square of the offset.",
    variable_names=Dict("a" => "theta", "b" => "offset"),
    prompts_dir="prompts/",
    api_key="token-abc123",
    model="meta-llama/Meta-Llama-3.1-8B-Instruct",
    api_kwargs=Dict("url" => "http://localhost:11440/v1"),
    verbose=true,
)

mach = machine(model, X, y)
fit!(mach)
report(mach)
predict(mach, X)
```

---

## Clone and Run Locally

After creating the 5D example, clone the LaSR repository to access the core implementation:

```bash
!git clone https://github.com/trishullab/LibraryAugmentedSymbolicRegression.jl.git
%cd LibraryAugmentedSymbolicRegression.jl
```

Then, run the script to train and evaluate the model:

```bash
!julia script.jl
```

Make sure the `script.jl` file is placed in the root of the cloned directory.

---

## File Structure

```
.
├── script.jl                # Main script to run LaSR
├── prompts/                 # Folder containing LLM prompt templates
└── outputs/                 # Output directory with logs and LLM call history
```

---

## Notes
- This setup assumes a local LLM server is running at `localhost:11440/v1`. Replace with your own endpoint if needed.
- Ensure the `prompts/` directory exists before running.

---

## Results
This project analyzes LaSR’s performance across varying input dimensions and reports structural similarity between predicted and ground truth formulas, even when exact coefficients differ slightly in high-dimensional cases.
