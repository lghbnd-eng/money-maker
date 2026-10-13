name: Python Package using Conda

on: [push]

jobs:
  build-linux:
    runs-on: ubuntu-latest
    strategy:
      max-parallel: 5

    steps:
      - uses: actions/checkout@v4

      - name: Set up Miniconda
        uses: conda-incubator/setup-miniconda@v3
        with:
          python-version: "3.10"
          auto-activate-base: true
          activate-environment: base

      - name: Install dependencies
        shell: bash -el {0}
        run: |
          if [ -f environment.yml ]; then
            conda env update --file environment.yml --name base --prune
          elif [ -f requirements.txt ]; then
            python -m pip install --upgrade pip
            pip install -r requirements.txt
          else
            echo "No environment.yml or requirements.txt found" >&2
            exit 1
          fi

      - name: Lint with flake8
        shell: bash -el {0}
        run: |
          conda install -y flake8
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics

      - name: Test with pytest
        shell: bash -el {0}
        run: |
          conda install -y pytest
          pytest
