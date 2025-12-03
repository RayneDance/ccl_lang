CCL Language Specification 1.5

Full Name: Composable Configuration Language

Extension: .ccl

MIME: application/ccl

1. Core Philosophy

CCL is designed to be human-readable first while enforcing type safety and reusability. It bridges the gap between loose formats (YAML/JSON) and strict configuration languages (CUE/Jsonnet).

2. General Syntax

2.1 Comments

Comments begin with # and continue to the end of the line.

Rule: Any character sequence starting with # that is not contained within a string literal is strictly treated as a comment.

# This is a comment
key: "value" # This is inline
url: "[http://example.com/#anchor](http://example.com/#anchor)" # This is NOT a comment (inside string)


2.2 Variables

Variables are mutable references defined using $.

Scope: Variables are file-scoped by default.

Export: Variables must be explicitly exported to be available to importers.

Assignment: $varName = value

$ver = 1.0
export $ver


3. Data Types

3.1 Reserved Keywords (Case Insensitive)

The following tokens have strict semantic meaning and cannot be used as unquoted strings:

True / False (Boolean)

None (Explicit Void/Null)

3.2 Numbers

Numbers are unencapsulated.

Integer: 123, -5

Float: 123.45, 1.0

The Zero Triad:

0 : The numeric integer zero.

-0 : Semantic "Empty Number". Represents "Auto", "Unlimited", or "Reset to Default".

None: Absence of value.

3.3 Strings

Strings must be encapsulated. Bare (unquoted) strings are invalid syntax, with one strict exception (see 4.2 Enums).

Double Quoted: "Hello World" (Supports standard escape sequences \n, \t, \").

Single Quoted: 'Hello World' (Literal string, minimal escaping).

Raw String: @"C:\Path\To\File" (Prefix with @. treats backslashes as literals. No escape sequences processed. Use "" to represent a double quote).

Multiline (Triple Quote Block):

Encapsulated by """.

The opening """ must be followed immediately by a Newline.

The closing """ must be on its own line.

Indentation: The indentation of the first line of content defines the baseline. All subsequent lines are trimmed relative to that baseline.

String Usage Guide

# 1. Double Quoted (Standard)
# Best for text requiring format control or Unicode.
greeting: "Hello\nWorld"       # Parsed as two lines
indented: "Column1\tColumn2"   # Tab separation
unicode: "Copyright \u00A9"    # Unicode escape sequences

# 2. Single Quoted (Literal)
# Best for simple text or HTML/XML where " is common.
title: 'Simple Title'
html_fragment: '<div class="box">'  # No need to escape double quotes

# 3. Raw Strings (Regex & Paths)
# Best for Regular Expressions and Windows File Paths.
# No escaping of backslashes required.
win_path: @"C:\Users\Admin\Documents"
email_regex: @"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$"
quoted_raw: @"She said, ""Hello!"""  # Use double "" for a quote

# 4. Multiline Strings
# Best for long descriptions or embedded content.
description: """
    This text is left-aligned.
    The indentation matches the start of "This".
"""


4. Structures & Enums

4.1 Struct Definition (~)

The struct definition uses the ~ symbol.

Syntax: ~ Name:

Inheritance: ~ Child(Parent):

Fields: name (type): default_value

4.2 Enum Definition (enum)

Enums must explicitly declare their backing type: (Int) or (String).

Auto Mode (Int): Values are auto-incremented integers starting at 0.

Auto Mode (String): The identifier itself becomes the string value. This is the only place in CCL where unencapsulated text is treated as a string.

Defined Mode: Explicit key-value mapping overrides auto behavior.

# Auto Mode (Int)
# Values: Left=0, Right=1, Up=2
enum Direction(Int):
    - Left
    - Right
    - Up

# Auto Mode (String)
# Values: "Debug", "Info", "Error"
# No quotes required for these identifiers.
enum LogLevel(String):
    - Debug
    - Info
    - Error

# Defined Mode (Mixed)
# Values: "administrator", "guest"
enum Roles(String):
    - Admin: "administrator"
    - Guest: "guest"


4.3 Syntax Disambiguation

~: Exclusively for Struct Definitions.

enum: Exclusively for Enumeration Definitions.

""": Exclusively for Multiline Strings.

4.4 Bare Implementation

It is not required to define a struct to create data. You may write "bare" lists and maps.

# Bare Map (Key-Value)
server:
    port: 8080
    host: "localhost"

# Bare List (Sequence)
regions:
    - "us-east-1"
    - "eu-west-1"

# Nested List of Maps
users:
    - name: "Alice"
      role: "admin"
    - name: "Bob"
      role: "editor"


4.5 Typed Implementation

To enforce schema validation, use the key (Type): syntax.

~ ServerConfig:
    port (int): 80
    host (string): "0.0.0.0"

# Instantiation
my_server (ServerConfig):
    port: 8080
    # 'host' uses default


5. Composition & Inheritance Rules

5.1 Inheritance

Inheritance performs a deep merge of the schema.

Parent fields are copied to the Child.

Overrides in the Child definition replace Parent defaults.

New fields are appended.

~ Base:
    id (int): 1

~ Extended(Base):
    name (string): "util"


5.2 Composition

Fields within a Struct can be typed as other Structs.

~ Database:
    driver: "postgres"

~ App:
    db (Database):
        driver: "mysql" # Override default during composition


6. Example Document

# global_config.ccl

$standard_timeout = 30

# 1. Type Definitions
# -------------------

# Auto Enum (Integers: 0, 1, 2)
enum Environment(Int):
    - Dev
    - Stage
    - Prod

# Auto Enum (Strings: "Admin", "Guest")
# Implicitly converts identifier to String value
enum Role(String):
    - Admin
    - Guest

~ Connection:
    retries (int): 3
    timeout (int): $standard_timeout
    env (Environment): Environment.Dev

~ HttpConnection(Connection):
    use_ssl (bool): True
    agent: "Mozilla"

# 2. Implementation
# -----------------

# Typed Object (Enforces HttpConnection schema)
main_gateway (HttpConnection):
    retries: 5
    # Using Enum Value (resolves to 2)
    env: Environment.Prod
    
    agent: "CustomBot/1.0"
    
    banner_msg: """
        Warning: Unauthorized access is prohibited.
        System ID: 442A
    """

# Using explicitly reserved words
feature_flags:
    beta_access: False   # Boolean
    legacy_mode: None    # Null type
