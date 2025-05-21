# Data Model: Musa Thought Leadership Engine

This data model outlines the core entities and their relationships required for the application.

## Entities

### 1. `User`
Represents an internal user of the application (Musa Capital team).

| Attribute          | Data Type     | Description                                      | Constraints / Notes                 |
|--------------------|---------------|--------------------------------------------------|-------------------------------------|
| `user_id`          | UUID/Integer  | Primary Key                                      | PK, Auto-increment or UUID          |
| `name`             | String        | User's full name                                 | Not Null                            |
| `email`            | String        | User's email address (for login)                 | Not Null, Unique                    |
| `password_hash`    | String        | Hashed password                                  | Not Null                            |
| `role`             | String        | E.g., "Admin", "Editor", "Partner"               | Enum                                |
| `created_at`       | Timestamp     | Timestamp of user creation                       | Not Null, Default CURRENT_TIMESTAMP |
| `updated_at`       | Timestamp     | Timestamp of last update                         | Not Null, Default CURRENT_TIMESTAMP |

### 2. `NewsSource`
Represents an industry news source to be scanned.

| Attribute         | Data Type     | Description                                      | Constraints / Notes                 |
|-------------------|---------------|--------------------------------------------------|-------------------------------------|
| `source_id`       | UUID/Integer  | Primary Key                                      | PK, Auto-increment or UUID          |
| `name`            | String        | Name of the news source (e.g., "Crunchbase")     | Not Null, Unique                    |
| `url_or_api_endpoint` | String    | Base URL, RSS feed URL, or API endpoint          | Not Null                            |
| `api_key`         | String        | API key for the source (if applicable)           | Encrypted, Nullable                 |
| `scan_frequency`  | String/Interval | How often to scan (e.g., "daily", "6 hours")   |                                     |
| `last_scanned_at` | Timestamp     | Timestamp of the last successful scan            | Nullable                            |
| `is_active`       | Boolean       | Whether the source is currently being scanned    | Not Null, Default TRUE              |
| `created_at`      | Timestamp     | Timestamp of source creation                     | Not Null, Default CURRENT_TIMESTAMP |
| `updated_at`      | Timestamp     | Timestamp of last update                         | Not Null, Default CURRENT_TIMESTAMP |

### 3. `NewsArticle`
Represents an individual news article retrieved from a `NewsSource`.

| Attribute         | Data Type     | Description                                      | Constraints / Notes                 |
|-------------------|---------------|--------------------------------------------------|-------------------------------------|
| `article_id`      | UUID/Integer  | Primary Key                                      | PK, Auto-increment or UUID          |
| `source_id`       | UUID/Integer  | Foreign Key to `NewsSource`                      | FK, Not Null                        |
| `title`           | String        | Title of the article                             | Not Null                            |
| `original_url`    | String        | URL of the original article                      | Not Null, Unique                    |
| `publication_date`| Timestamp     | Date the article was published                   | Nullable                            |
| `retrieved_at`    | Timestamp     | Date the article was retrieved by our system     | Not Null, Default CURRENT_TIMESTAMP |
| `summary`         | Text          | Short summary of the article (from source or AI) | Nullable                            |
| `full_content`    | Text          | Full text content of the article (if stored)     | Nullable                            |
| `keywords`        | Array[String] | Extracted keywords                               | Nullable                            |
| `relevance_score` | Float         | Score based on Musa's criteria (if calculated)   | Nullable                            |
| `processed_at`    | Timestamp     | Timestamp when AI processing/linking was done    | Nullable                            |

### 4. `LumaEvent`
Represents an event from Musa Capital's Luma calendar.

| Attribute         | Data Type     | Description                                   | Constraints / Notes                 |
|-------------------|---------------|-----------------------------------------------|-------------------------------------|
| `event_id`        | UUID/Integer  | Primary Key                                   | PK, Auto-increment or UUID          |
| `luma_native_id`  | String        | Original ID from Luma (for syncing)           | Not Null, Unique                    |
| `title`           | String        | Title of the event                            | Not Null                            |
| `description`     | Text          | Description of the event                      | Nullable                            |
| `event_date_time` | Timestamp     | Date and time of the event                    | Not Null                            |
| `luma_url`        | String        | URL to the event page on Luma                 | Not Null                            |
| `target_audience` | String        | Intended audience for the event               | Nullable                            |
| `topics`          | Array[String] | Keywords/topics associated with the event     | Nullable                            |
| `last_synced_at`  | Timestamp     | Timestamp of last sync with Luma              | Not Null, Default CURRENT_TIMESTAMP |
| `created_at`      | Timestamp     | Timestamp of event record creation            | Not Null, Default CURRENT_TIMESTAMP |

### 5. `ContentIdea` (or `TopicSuggestion`)
Represents a potential thought leadership topic linking a `NewsArticle` to a `LumaEvent`.

| Attribute         | Data Type     | Description                                      | Constraints / Notes                 |
|-------------------|---------------|--------------------------------------------------|-------------------------------------|
| `idea_id`         | UUID/Integer  | Primary Key                                      | PK, Auto-increment or UUID          |
| `news_article_id` | UUID/Integer  | Foreign Key to `NewsArticle`                     | FK, Not Null                        |
| `luma_event_id`   | UUID/Integer  | Foreign Key to `LumaEvent` (can be nullable if idea is not event-tied) | FK, Nullable        |
| `suggested_angle` | Text          | AI-suggested angle or key talking points         | Nullable                            |
| `relevance_score` | Float         | Score indicating how well the news and event align | Nullable                            |
| `status`          | String        | E.g., "suggested", "reviewing", "rejected", "drafting" | Enum, Not Null, Default "suggested" |
| `created_by_user_id` | UUID/Integer | User who might have manually created/flagged this | FK to `User`, Nullable              |
| `created_at`      | Timestamp     | Timestamp of idea creation                       | Not Null, Default CURRENT_TIMESTAMP |
| `updated_at`      | Timestamp     | Timestamp of last update                         | Not Null, Default CURRENT_TIMESTAMP |

### 6. `ThoughtPiece` (or `GeneratedContent`)
Represents the actual generated thought leadership content.

| Attribute        | Data Type     | Description                                         | Constraints / Notes                 |
|------------------|---------------|-----------------------------------------------------|-------------------------------------|
| `piece_id`       | UUID/Integer  | Primary Key                                         | PK, Auto-increment or UUID          |
| `idea_id`        | UUID/Integer  | Foreign Key to `ContentIdea`                        | FK, Not Null (or directly to article/event if no Idea entity) |
| `title`          | String        | Headline for the thought piece                      | Not Null                            |
| `ai_draft_content` | Text        | Initial draft generated by AI                       | Nullable                            |
| `final_content_markdown` | Text    | User-edited and finalized content in Markdown     | Nullable                            |
| `final_content_html` | Text      | User-edited and finalized content in HTML         | Nullable                            |
| `status`         | String        | E.g., "draft", "review", "approved", "published"    | Enum, Not Null, Default "draft"     |
| `created_by_user_id` | UUID/Integer | User who initiated generation or is primary author | FK to `User`, Not Null             |
| `reviewed_by_user_id` | UUID/Integer | User who reviewed/approved the piece            | FK to `User`, Nullable              |
| `published_at`   | Timestamp     | Timestamp when the piece was published              | Nullable                            |
| `target_platform`| String        | E.g., "LinkedIn", "Newsletter", "Both"            | Enum                                |
| `linkedin_url`   | String        | URL if published to LinkedIn                        | Nullable                            |
| `newsletter_id`  | String        | Identifier if included in a specific newsletter     | Nullable                            |
| `created_at`     | Timestamp     | Timestamp of piece creation                         | Not Null, Default CURRENT_TIMESTAMP |
| `updated_at`     | Timestamp     | Timestamp of last update                            | Not Null, Default CURRENT_TIMESTAMP |

### 7. `MusaConfiguration`
Stores global settings and prompts for the AI.

| Attribute      | Data Type     | Description                                       | Constraints / Notes                 |
|----------------|---------------|---------------------------------------------------|-------------------------------------|
| `config_id`    | UUID/Integer  | Primary Key                                       | PK, Auto-increment or UUID          |
| `setting_key`  | String        | Name of the setting (e.g., "musa_preseed_perspective", "tone_of_voice_guidelines") | Not Null, Unique |
| `setting_value`| Text          | The actual value of the setting/prompt            | Not Null                            |
| `description`  | Text          | Description of what the setting is for            | Nullable                            |
| `is_active`    | Boolean       | Whether this configuration is currently in use    | Not Null, Default TRUE              |
| `updated_at`   | Timestamp     | Timestamp of last update                          | Not Null, Default CURRENT_TIMESTAMP |
| `updated_by_user_id`| UUID/Integer | User who last updated this config              | FK to `User`, Nullable              |

---

## Relationships

*   **One-to-Many:**
    *   `User` has many `ThoughtPiece` (created_by, reviewed_by).
    *   `User` has many `ContentIdea` (created_by).
    *   `User` has many `MusaConfiguration` (updated_by).
    *   `NewsSource` has many `NewsArticle`.
    *   `NewsArticle` can be part of many `ContentIdea` (a single news item might inspire multiple angles/event tie-ins).
    *   `LumaEvent` can be part of many `ContentIdea`.
    *   `ContentIdea` can lead to one `ThoughtPiece` (typically, though one could argue for multiple variations).

*   **Many-to-Many (Conceptual, often resolved with join tables if needed, but here represented by FKs in `ContentIdea`):**
    *   A `NewsArticle` can be linked with multiple `LumaEvent`s through `ContentIdea`.
    *   A `LumaEvent` can be linked with multiple `NewsArticle`s through `ContentIdea`.

---

## Notes/Considerations:

*   **Data Types:** Specific data types (e.g., `UUID` vs `Integer` for PKs) will depend on the chosen database system.
*   **Indexes:** Critical foreign keys and frequently queried fields (e.g., `original_url` in `NewsArticle`, `status` fields) should be indexed for performance.
*   **Normalization:** This model aims for a reasonable level of normalization. Denormalization might be considered for specific performance-critical queries later.
*   **AI Prompts:** The `MusaConfiguration` entity is crucial for storing reusable prompts and guidelines that define Musa Capital's voice and perspective for the AI.
*   **Flexibility:** The `ContentIdea` entity provides flexibility in managing potential topics before committing to generating full content. If a simpler flow is desired initially, `ThoughtPiece` could directly link to `NewsArticle` and `LumaEvent`.
*   **Error Handling/Logging:** While not explicitly part of the data model, tables for logging API calls, errors, and system events would be necessary.
