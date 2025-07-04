This file is a merged representation of the entire codebase, combining all repository files into a single document.
Generated by Repomix on: 2025-05-23 22:04:55

# File Summary

## Purpose:

This file contains a packed representation of the entire repository's contents.
It is designed to be easily consumable by AI systems for analysis, code review,
or other automated processes.

## File Format:

The content is organized as follows:
1. This summary section
2. Repository information
3. Repository structure
4. Multiple file entries, each consisting of:
   a. A header with the file path (## File: path/to/file)
   b. The full contents of the file in a code block

## Usage Guidelines:

- This file should be treated as read-only. Any changes should be made to the
  original repository files, not this packed version.
- When processing this file, use the file path to distinguish
  between different files in the repository.
- Be aware that this file may contain sensitive information. Handle it with
  the same level of security as you would the original repository.

## Notes:

- Some files may have been excluded based on .gitignore rules and Repomix's
  configuration.
- Binary files are not included in this packed representation. Please refer to
  the Repository Structure section for a complete list of file paths, including
  binary files.

## Additional Information:

For more information about Repomix, visit: https://github.com/andersonby/python-repomix


# Repository Structure

```
.gitignore
project_inspector.py
README.md
requirements.txt
```

# Repository Files


## .gitignore

```text
__pycache__/
*.pyc
*.pyo
*.pyd
.env
.venv/
venv/
dist/
build/
*.egg-info/
.pytest_cache/
.aider*
```

## project_inspector.py

```python
#!/usr/bin/env python3
"""
Project Inspector - A tool to analyze project structure and provide insights.
"""

import argparse
import os
import subprocess
import sys
import json
from datetime import datetime
from typing import Dict, Any, Optional

def parse_arguments() -> argparse.Namespace:
    """Parse command line arguments."""
    parser = argparse.ArgumentParser(
        description="Analyze project structure and provide insights.",
        formatter_class=argparse.ArgumentDefaultsHelpFormatter
    )
    
    parser.add_argument(
        "directory", 
        help="Path to the project directory to analyze"
    )
    
    parser.add_argument(
        "--output", "-o",
        help="Output file to save the report (default: stdout)",
        default=None
    )
    
    parser.add_argument(
        "--verbose", "-v",
        action="store_true",
        help="Enable verbose output"
    )
    
    return parser.parse_args()

def check_directory(directory: str) -> bool:
    """
    Check if the specified directory exists.
    
    Args:
        directory: Path to the directory to check
        
    Returns:
        bool: True if directory exists, False otherwise
    """
    if not os.path.exists(directory):
        print(f"Error: Directory '{directory}' does not exist.", file=sys.stderr)
        return False
    
    if not os.path.isdir(directory):
        print(f"Error: '{directory}' is not a directory.", file=sys.stderr)
        return False
    
    return True

def extract_context(directory: str, verbose: bool = False) -> Optional[str]:
    """
    Extract code context using repomix.
    
    Args:
        directory: Path to the project directory
        verbose: Whether to show verbose output
        
    Returns:
        str or None: Extracted context as text, or None if extraction failed
    """
    try:
        cmd = ["repomix", "--stdout", directory]
        
        if verbose:
            print(f"Running: {' '.join(cmd)}")
            
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            check=True
        )
        
        return result.stdout
    except subprocess.CalledProcessError as e:
        print(f"Error running repomix: {e}", file=sys.stderr)
        if verbose:
            print(f"stderr: {e.stderr}", file=sys.stderr)
        return None
    except FileNotFoundError:
        print("Error: 'repomix' command not found. Please install it first.", file=sys.stderr)
        return None

def analyze_context(context: str, verbose: bool = False) -> Optional[Dict[str, Any]]:
    """
    Analyze the extracted context using an LLM.
    
    Args:
        context: The extracted context as text
        verbose: Whether to show verbose output
        
    Returns:
        Dict or None: Analysis results as a dictionary, or None if analysis failed
    """
    try:
        cmd = ["llm", "analyze", "--format", "json"]
        
        if verbose:
            print(f"Running: {' '.join(cmd)}")
            
        result = subprocess.run(
            cmd,
            input=context,
            capture_output=True,
            text=True,
            check=True
        )
        
        return json.loads(result.stdout)
    except subprocess.CalledProcessError as e:
        print(f"Error running LLM analysis: {e}", file=sys.stderr)
        if verbose:
            print(f"stderr: {e.stderr}", file=sys.stderr)
        return None
    except json.JSONDecodeError:
        print("Error: Failed to parse LLM output as JSON", file=sys.stderr)
        return None
    except FileNotFoundError:
        print("Error: 'llm' command not found. Please install it first.", file=sys.stderr)
        return None

def format_report(directory: str, context: str, analysis: Dict[str, Any]) -> str:
    """
    Format the analysis results into a readable report.
    
    Args:
        directory: The analyzed directory path
        context: The extracted context
        analysis: The analysis results
        
    Returns:
        str: Formatted report
    """
    report = []
    
    # Header
    report.append("=" * 80)
    report.append(f"PROJECT INSPECTOR REPORT - {os.path.basename(os.path.abspath(directory))}")
    report.append(f"Generated on: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    report.append("=" * 80)
    report.append("")
    
    # Project overview
    report.append("PROJECT OVERVIEW")
    report.append("-" * 80)
    
    if "project_summary" in analysis:
        report.append(analysis["project_summary"])
    else:
        report.append("No project summary available.")
    
    report.append("")
    
    # File statistics
    report.append("FILE STATISTICS")
    report.append("-" * 80)
    
    # Since context is now text, we can't access it as a dictionary
    report.append("Context extracted with repomix")
    
    report.append("")
    
    # Recommendations
    report.append("RECOMMENDATIONS")
    report.append("-" * 80)
    
    if "recommendations" in analysis:
        for i, rec in enumerate(analysis["recommendations"], 1):
            report.append(f"{i}. {rec}")
    else:
        report.append("No recommendations available.")
    
    report.append("")
    report.append("=" * 80)
    
    return "\n".join(report)

def main():
    """Main entry point for the script."""
    args = parse_arguments()
    
    # Check if directory exists
    if not check_directory(args.directory):
        sys.exit(1)
    
    print(f"Analyzing project in '{args.directory}'...")
    
    # Extract context
    context = extract_context(args.directory, args.verbose)
    if not context:
        sys.exit(1)
    
    print("Context extraction complete.")
    
    # Analyze context
    print("Analyzing project structure...")
    analysis = analyze_context(context, args.verbose)
    if not analysis:
        sys.exit(1)
    
    # Format and output report
    report = format_report(args.directory, context, analysis)
    
    if args.output:
        try:
            with open(args.output, 'w') as f:
                f.write(report)
            print(f"Report saved to '{args.output}'")
        except IOError as e:
            print(f"Error writing to output file: {e}", file=sys.stderr)
            sys.exit(1)
    else:
        print("\n" + report)
    
    print("Analysis complete!")

if __name__ == "__main__":
    main()
```

## README.md

```markdown

```

## requirements.txt

```text

```

## Statistics

- Total Files: 4
- Total Characters: 6473
- Total Tokens: 0
