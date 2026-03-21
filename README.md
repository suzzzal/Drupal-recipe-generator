## 1. User Prompt

### Overview

The **User Prompt** step is the entry point of the system, where users describe the desired Drupal feature in natural language. This input is later transformed into a structured prompt for AI-based recipe generation.

To ensure clarity and usability, this step is implemented using a guided interface built with Drupal Form API.

---

### User Interface Design

The interface is designed to be simple, intuitive, and helpful for both technical and non-technical users.

#### 🔹 Prompt Input Area
- A large textarea where users can describe their requirements in natural language  
- Includes placeholder text (e.g., *“Create a Team Member content type with photo and bio fields…”*)  
- Supports multi-line input for detailed descriptions  

#### 🔹 Feature Hints
- Displays commonly used Drupal concepts to guide users  
- Includes:
  - Content Types  
  - Fields (text, image, date, etc.)  
  - Views (grid, list, table)  
  - Taxonomy (tags, categories)  
  - Media  

This helps users structure their prompts more effectively.

#### 🔹 Example Prompts
- Provides clickable sample prompts such as:
  - “Create a blog content type with title and body”  
  - “Create an event system with date and calendar view”  
- Clicking an example auto-fills the input field  

#### 🔹 Action Button
- A clear call-to-action button: **“Generate Recipe”**  
- Submits the prompt and triggers the AI generation pipeline  

#### 🔹 Optional Enhancements
- Autocomplete suggestions while typing  
- Feature chips (clickable keywords like *Field*, *View*, *Image*)  
- Basic validation for empty or unclear input  

---

### Example: User Prompt Form (Drupal Form API)

```php








class RecipeGeneratorForm extends FormBase {

  public function __construct(protected $aiRecipeGenerator) {}

  public static function create(ContainerInterface $container) {
    return new static($container->get('ai_recipe.generator'));
  }

  public function getFormId() {
    return 'recipe_generator_form';
  }

  public function buildForm(array $form, FormStateInterface $form_state) {
    $form['prompt'] = ['#type' => 'textarea', '#title' => $this->t('Describe feature'), '#required' => TRUE];
    $form['submit'] = ['#type' => 'submit', '#value' => $this->t('Generate Recipe')];
    return $form;
  }

  public function submitForm(array &$form, FormStateInterface $form_state) {
    $this->aiRecipeGenerator->generate($form_state->getValue('prompt'));
    $this->messenger()->addMessage($this->t('Recipe generated successfully!'));
  }
}
  public function submitForm(array &$form, FormStateInterface $form_state) {
    $prompt = $form_state->getValue('prompt');

    $this->aiRecipeGenerator->generate($prompt);
    
    $this->messenger()->addMessage($this->t('Recipe generated successfully!'));
  }
}

----
class RecipeGeneratorForm extends FormBase {

  public function buildForm(array $form, FormStateInterface $form_state) {
    $form['prompt'] = [
      '#type' => 'textarea',
      '#title' => 'Describe the feature',
      '#required' => TRUE,
    ];

    $form['submit'] = [
      '#type' => 'submit',
      '#value' => 'Generate Recipe',
    ];

    return $form;
  }

  public function submitForm(array &$form, FormStateInterface $form_state) {
    $prompt = $form_state->getValue('prompt');

    \Drupal::service('ai_recipe.generator')->generate($prompt);
  }
}

class SiteConfigService {

  public function getSummary() {
    $content_types = \Drupal::entityTypeManager()
      ->getStorage('node_type')
      ->loadMultiple();

    $fields = \Drupal::entityTypeManager()
      ->getStorage('field_storage_config')
      ->loadMultiple();

    return [
      'content_types' => array_keys($content_types),
      'fields' => array_keys($fields),
    ];
  }
}

$data = Yaml::parse($yaml);
if (!is_array($data)) throw new Exception("Invalid YAML");


// Schema check
if (empty($config['name']) || empty($config['install'])) {
  throw new Exception("Invalid schema");
}

// Conflict check
if (loadContentType($config['type'])) {
  throw new Exception("Content type exists");
}


while ($attempts < 3) {
  try { return generate($prompt); }
  catch ($e) { $prompt .= "Fix: ".$e->getMessage(); }
}
throw new Exception("Failed after retries");

// Basic schema validation
if (empty($config['name']) || empty($config['install']) || empty($config['type'])) {
  throw new Exception("Invalid configuration schema");
}

// Basic Conflict check: content type already exists
$existing = \Drupal::entityTypeManager()
  ->getStorage('node_type')
  ->load($config['type']);

if ($existing) {
  throw new Exception("Content type already exists");
}

// Validate allowed modules
$allowed = ['node', 'views', 'image', 'taxonomy'];
if (array_diff($config['install'], $allowed)) {
  throw new Exception("Invalid module dependency");
}


$zip = new \ZipArchive();
$zip->open('recipe.zip', \ZipArchive::CREATE);

$zip->addFromString('recipe.yml', $recipe_yaml);

foreach ($config_files as $name => $content) {
  $zip->addFromString("config/$name", $content);
}

$zip->close();


```
### Flow

User opens Recipe Generator UI
↓
Enters prompt (natural language)
↓
(Optional) Uses hints/examples
↓
Submits request


---

### UX Goals

- Guide users to write structured and meaningful prompts  
- Reduce ambiguity in natural language input  
- Improve the quality and accuracy of AI-generated recipes  
- Provide a smooth and beginner-friendly experience  

---

### Suggested Visualization

----------------------------------------
| Describe your feature                |
| [______________________________]     |
|                                      |
| Hints: Content Type | Field | View   |
|                                      |
| Examples:                            |
| - Blog system                        |
| - Event calendar                     |
|                                      |
|            [ Generate Recipe ]       |
---------------------------------------- |


