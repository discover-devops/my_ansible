# YAML (YAML Ain't Markup Language)

## Introduction

YAML, which stands for "YAML Ain't Markup Language" or sometimes recursively as "YAML Ain't Markup Language," is a human-readable data serialization format. It is often used for configuration files and data exchange between languages with different data structures. YAML is particularly prevalent in the context of configuration management tools like Ansible and Kubernetes.

## Basics of YAML

### Key-Value Pairs

In YAML, data is represented using key-value pairs. The key and value are separated by a colon, and the pairs are defined with spaces:

```yaml
city: Varanasi
month: May
year: 2022
food: Curry
```

### Arrays/Lists

Arrays or lists in YAML are denoted by a hyphen (-) followed by a space. This indicates the beginning of an element in the list:

```yaml
city:
  - Varanasi
  - Lucknow
  - New Delhi

food:
  - Roti
  - Curry
  - Daal
```

### Dictionaries/Maps

Dictionaries or maps in YAML are represented by key-value pairs grouped under an item. The indentation is crucial, and an equal number of spaces must be maintained for each property:

```yaml
curry:
  calories: 100
  vitamins: 20
  carbs: 20

banana:
  calories: 70
  vitamins: 50
  carbs: 20
```

### Dictionary of Lists

Combining dictionaries and lists provides flexibility in structuring data:

```yaml
city:
  - newdelhi:
      population: 100
      male: 65
      female: 35
  - varanasi:
      population: 75
      male: 55
      female: 45
```

### Dictionary vs. List vs. List of Dictionary

- **Dictionary (Map):**
  - Used to store different properties of a single object.
  - Can contain nested dictionaries.

- **List (Array):**
  - Used to store multiple items of the same type.
  - Ordered collection.

- **List of Dictionary:**
  - Combines the features of both dictionaries and lists.
  - Allows for a structured representation of multiple items.

## YAML Syntax and Formatting

- YAML is an unordered collection (for dictionaries/maps) and an ordered collection (for arrays/lists).
- The space or indentation is crucial in YAML and defines the structure.
- Consistent indentation is required for readability and proper interpretation.

## Resources

- [YAML Syntax Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)
- [Online YAML Linting](http://www.yamllint.com/)

YAML provides a human-readable and straightforward way to represent data, making it a popular choice for configuration files and data exchange in various domains.

---

