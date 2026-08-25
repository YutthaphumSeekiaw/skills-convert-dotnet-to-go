# .NET-to-Go Conversion

This context defines the vocabulary for converting selected .NET API routes into a reusable Go service while preserving the public contract and recording decisions.

## Conversion

**Source API**:
The .NET API codebase being analyzed as the behavioral source.
_Avoid_: legacy API, old service

**Target service**:
The Go codebase produced from the selected Source API routes using a Go template.
_Avoid_: Go rewrite, destination API

**Route**:
One HTTP operation identified by `operationId`, or by normalized HTTP method and path when no operationId exists.
_Avoid_: endpoint, action, controller

**Target datastore**:
The persistence system selected for one Route, either the existing SQL Server or a new datastore supported by the Target service.
_Avoid_: Go database, destination database

**Template profile**:
The structured description of the Go template's layout, framework, persistence, configuration, testing, and validation conventions.
_Avoid_: boilerplate, starter project

## Evidence and Delivery

**Conversion plan**:
An approved set of Route decisions, mappings, dependencies, risks, and validation criteria.
_Avoid_: migration checklist, implementation notes

**Blocker**:
An unresolved conflict, unsupported behavior, missing evidence, or failed validation that prevents a Route from being converted safely.
_Avoid_: TODO, warning

**Parity fixture**:
A recorded request and expected observable response and side effects used to compare the Source API and Target service.
_Avoid_: snapshot, sample payload

**Parity report**:
The evidence showing whether a converted Route matches its approved contract and observable behavior.
_Avoid_: conversion summary

**Conversion manifest**:
The machine-readable record of inputs, fingerprints, versions, decisions, artifacts, and validation results needed to resume or reproduce a conversion.
_Avoid_: config file, state file
