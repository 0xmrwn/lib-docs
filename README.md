# lib-docs

A curated collection of technical documentation extracted and compiled for LLM context usage.

## Purpose

This repository serves as a centralized store for documentation from various libraries and tools, extracted in markdown format and organized for easy consumption by LLMs. The goal is to provide clean, structured documentation that can be used as context when working with these technologies.

## Workflow

Documentation is extracted using [Firecrawl](https://firecrawl.dev/) to convert web-based documentation into clean markdown format. The extracted content is then organized by library/tool and stored in dedicated directories for easy reference and LLM context injection.

## Structure

- `code-server/` - Documentation for code-server
- `langchain/` - Documentation for LangChain
- Additional directories are added as new libraries are documented

## Usage

The markdown files in this repository are designed to be consumed programmatically as context for LLM interactions, enabling better assistance with library-specific questions and implementation guidance.
