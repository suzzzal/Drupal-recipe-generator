# Step 4 – AI Layer Implementation

## Overview

The AI Layer is the core intelligence of the system, responsible for transforming structured prompts into valid, schema-compliant Drupal Recipe configurations. Unlike traditional text generation systems, this layer is designed as a **context-aware configuration engine** that understands Drupal-specific structures, dependencies, and constraints.

This layer does not operate in isolation. It works in coordination with:

* Prompt Engineering Layer (input structuring)
* Fetching Layer (site context awareness)
* Validation Layer (schema + conflict enforcement)

The goal is to ensure that the generated output is not just syntactically correct, but also **semantically valid within a Drupal environment**.

---

## Key Responsibilities of the AI Layer

The AI system performs multiple responsibilities simultaneously:

### 1. Understanding User Intent

* Interprets natural language requirements
* Maps user needs to Drupal entities (content types, fields, views)

### 2. Context Awareness

* Uses injected site configuration
* Avoids duplication of existing structures
* Aligns with current site architecture

### 3. Structured YAML Generation

* Generates `recipe.yml` and related configuration files
* Maintains correct hierarchy and formatting

### 4. Schema Alignment

* Ensures generated configuration follows Drupal standards
* Uses predefined schema rules

### 5. Dependency Resolution

* Automatically identifies required modules
* Adds correct dependencies in `install` section

### 6. Multi-file Generation

* Produces multiple YAML files (content type, field storage, views, etc.)
* Maintains correct file naming conventions

### 7. Error Correction

* Fixes invalid outputs during retry loops
* Uses validation feedback to regenerate YAML

### 8. Conflict Awareness

* Avoids overwriting existing configurations
* Suggests reuse or renaming strategies

### 9. Format Enforcement

* Ensures strict YAML-only output
* Prevents additional explanations or noise

### 10. Model Adaptation

* Supports multiple LLM providers (Claude, GPT, Gemini)
* Handles fallback logic dynamically

---

## Architecture of AI Layer

```
User Prompt
    ↓
Structured Prompt (Prompt Engineering)
    ↓
AI Layer (LLM)
    ↓
Generated YAML
    ↓
Parser + Validation
    ↓
Retry Loop (if needed)
    ↓
Final Output
```

---

## LLM Integration via Drupal AI Module

The system integrates with the Drupal AI module to interact with multiple LLM providers.

### Service Example

```php
class AIService {

  protected $provider;

  public function __construct($provider) {
    $this->provider = $provider;
  }

  public function generate($prompt) {
    $ai = \Drupal::service('ai.provider')
      ->setProvider($this->provider);

    return $ai->generate($prompt);
  }
}
```

---

## Model Fallback Strategy

To improve reliability, the system implements fallback logic across multiple models.

```php
class LLMService {

  public function generate($prompt) {
    $providers = ['claude', 'gpt', 'gemini'];

    foreach ($providers as $provider) {
      try {
        $ai = \Drupal::service('ai.provider')
          ->setProvider($provider);

        $response = $ai->generate($prompt);

        if (!empty($response)) {
          return $response;
        }
      } catch (\Exception $e) {
        continue;
      }
    }

    throw new \Exception("All LLM providers failed.");
  }
}
```

---

## Prompt Execution Strategy

The AI Layer receives a **structured prompt** composed of:

* System Instructions
* Rules & Constraints
* Drupal Context
* Examples
* User Request

Example prompt:

```text
You are a Drupal configuration expert.

Generate Drupal Recipe YAML.

Rules:
- Output only YAML
- Follow Drupal schema
- No explanations

Existing content types:
article, page

User request:
Create a Team Member content type with photo and bio fields.

Output:
Valid Drupal recipe YAML files.
```

---

## Multi-file YAML Generation

The AI is expected to generate:

* `recipe.yml`
* `node.type.team_member.yml`
* `field.storage.node.field_photo.yml`
* `field.field.node.team_member.field_photo.yml`
* `views.view.team_members.yml`

Example:

```yaml
name: Team Member Recipe
type: recipe
description: Adds a Team Member content type

install:
  - node
  - image
  - views
```

---

## Dependency Resolution Logic

The AI automatically determines required modules:

```php
function resolveDependencies($config) {
  $dependencies = [];

  if (isset($config['fields'])) {
    $dependencies[] = 'field';
  }

  if (isset($config['image_fields'])) {
    $dependencies[] = 'image';
  }

  if (isset($config['views'])) {
    $dependencies[] = 'views';
  }

  return array_unique($dependencies);
}
```

---

## Schema Alignment Strategy

The AI is guided to follow Drupal schema patterns such as:

* `node.type.*`
* `field.storage.*`
* `field.field.*`
* `views.view.*`

Example validation:

```php
function validateSchema($config) {
  if (!isset($config['name'])) {
    throw new \Exception("Missing recipe name");
  }

  if (!isset($config['install'])) {
    throw new \Exception("Missing install dependencies");
  }

  return TRUE;
}
```

---

## Error Correction & Regeneration

The AI Layer works with a retry mechanism:

```php
class RetryService {

  public function generateWithRetry($prompt, $llm) {
    $attempts = 0;

    while ($attempts < 3) {
      $yaml = $llm->generate($prompt);

      try {
        validateSchema(Yaml::parse($yaml));
        return $yaml;
      } catch (\Exception $e) {
        $prompt .= "\nFix error: " . $e->getMessage();
        $attempts++;
      }
    }

    throw new \Exception("Failed after retries");
  }
}
```

---

## Conflict Awareness

The AI considers existing site configuration:

```php
function checkConflict($type) {
  $existing = \Drupal::entityTypeManager()
    ->getStorage('node_type')
    ->load($type);

  if ($existing) {
    throw new \Exception("Content type already exists");
  }
}
```

---

## Format Enforcement

The system ensures:

* YAML-only output
* No explanations
* Proper indentation

Example enforcement:

```php
function enforceFormat($output) {
  if (strpos($output, ':') === FALSE) {
    throw new \Exception("Invalid YAML format");
  }

  return $output;
}
```

---

## Performance Considerations

* Limit retries to 3–5 attempts
* Cache repeated prompts
* Minimize token usage with summarized context

---

## Challenges & Mitigation

| Challenge           | Solution                       |
| ------------------- | ------------------------------ |
| Hallucinated YAML   | Schema validation + retry loop |
| Duplicate config    | Context injection              |
| Invalid syntax      | YAML parser                    |
| Model inconsistency | Multi-LLM fallback             |

---

## Key Design Principle

> The AI is treated as an assistant, not a source of truth.
> All outputs are validated, sanitized, and verified before use.

---

## Conclusion

The AI Layer transforms the system from a simple text generator into a **reliable configuration engine** capable of producing production-ready Drupal Recipes. By combining structured prompts, context awareness, validation pipelines, and multi-model support, the system ensures high-quality, consistent, and usable outputs.

This layer is critical in bridging the gap between natural language input and structured Drupal configuration, enabling a scalable and user-friendly approach to recipe generation.
