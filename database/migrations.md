# 마이그레이션

## 마이그레이션 파일 위치

`htdocs/database/migrations/`

## 마이그레이션 실행

```bash
cd htdocs

# 모든 마이그레이션 실행
php artisan migrate

# 마이그레이션 상태 확인
php artisan migrate:status

# 마이그레이션 롤백 (마지막 배치)
php artisan migrate:rollback

# 모든 마이그레이션 롤백 후 재실행
php artisan migrate:fresh

# 시드 데이터와 함께 재실행
php artisan migrate:fresh --seed
```

## 마이그레이션 히스토리

### 2025-07-23: 핵심 시스템

```
2025_07_23_122850_create_company_table.php
2025_07_23_124011_create_users_table.php
2025_07_23_125351_create_personal_access_tokens_table.php
2025_07_23_164654_add_active_to_companies_table.php
2025_07_23_172147_create_test_types_table.php
```

### 2025-07-24: 인성검사 시스템

```
2025_07_24_044509_create_personality_codes_table.php
2025_07_24_052745_add_active_to_personality_codes_table.php
2025_07_24_060222_create_personality_questions_table.php
2025_07_24_061152_add_active_to_personality_questions_table.php
2025_07_24_080221_add_description_to_personality_test_types_table.php
2025_07_24_105312_create_personality_sheets_table.php
2025_07_24_130758_add_problem_order_to_personality_sheets_table.php
2025_07_24_162237_create_personalities_table.php
2025_07_24_162237_create_personality_applicants_table.php
```

### 2025-08-07~09: 기본값 및 캐시

```
2025_08_07_173614_add_default_value_to_answer_field.php
2025_08_07_173711_add_default_values_to_personality_applicants_table.php
2025_08_09_092043_create_cache_table.php
2025_08_09_092043_create_sessions_table.php
```

### 2025-08-17~18: 평판진단 및 다면평가

```
2025_08_17_143326_create_reputation_codes_table.php
2025_08_17_143332_create_reputation_questions_table.php
2025_08_18_032634_create_reputation_tasks_table.php
2025_08_18_034117_create_reputation_applicants_table.php
2025_08_18_034200_create_reputation_raters_table.php
2025_08_18_071531_create_multifaceted_test_types_table.php
2025_08_18_071559_create_multifaceted_codes_table.php
2025_08_18_071618_create_multifaceted_questions_table.php
2025_08_18_090000_create_multifaceted_applicants_table.php
2025_08_18_091000_create_multifaceted_raters_table.php
2025_08_18_161126_add_display_code_to_personality_sheets_table.php
```

### 2025-08-26~29: 통계 및 채점

```
2025_08_26_194100_add_email_sent_at_to_reputation_raters_table.php
2025_08_27_144029_add_top_percentiles_to_personality_data_statistics.php
2025_08_27_144255_update_personality_data_statistics_percentiles.php
2025_08_27_150400_add_metadata_fields_to_personality_data_statistics.php
2025_08_27_150832_add_statistics_reflected_field.php
2025_08_28_214311_add_report_value_to_personality_applicants_table.php
2025_08_28_235211_add_start_date_to_multifaceted_applicants_table.php
2025_08_29_031019_add_contact_fields_to_raters_tables.php
2025_08_29_054523_add_scoring_fields_to_raters_tables.php
```

### 2025-09: 평판 데이터 및 공지사항

```
2025_09_03_192508_add_scoring_record_to_raters_tables.php
2025_09_03_225720_add_statistics_updated_to_reputation_raters.php
2025_09_03_225916_update_reputation_data_foreign_keys.php
2025_09_03_230010_fix_company_idx_to_bigint.php
2025_09_14_164612_create_notices_table.php
2025_09_15_001526_add_scoring_status_to_personality_applicants.php
2025_09_15_011220_add_scoring_status_to_personality_applicants_table.php
```

### 2025-10~2026-01: 추가 기능

```
2025_10_11_233746_add_login_count_to_applicants_tables.php
2025_01_27_000000_add_scoring_record_to_multifaceted_applicants.php
2025_01_28_000000_create_personality_grade_criteria_table.php
2026_01_18_013621_create_report_access_tokens_table.php
```

## 마이그레이션 작성 예시

### 새 테이블 생성

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('example_table', function (Blueprint $table) {
            $table->id();
            $table->uuid('idx')->unique();
            $table->string('name');
            $table->foreignId('company_id')->constrained();
            $table->json('data')->nullable();
            $table->boolean('active')->default(true);
            $table->timestamps();

            $table->index('company_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('example_table');
    }
};
```

### 컬럼 추가

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('existing_table', function (Blueprint $table) {
            $table->string('new_column')->nullable()->after('existing_column');
            $table->boolean('is_active')->default(false);
        });
    }

    public function down(): void
    {
        Schema::table('existing_table', function (Blueprint $table) {
            $table->dropColumn(['new_column', 'is_active']);
        });
    }
};
```

### 인덱스 추가

```php
Schema::table('personality_applicants', function (Blueprint $table) {
    $table->index('personality_idx');
    $table->index(['company_id', 'is_complete']);
});
```

## 마이그레이션 생성 명령어

```bash
# 새 테이블 생성
php artisan make:migration create_example_table

# 기존 테이블 수정
php artisan make:migration add_column_to_example_table --table=example

# 모델과 함께 생성
php artisan make:model Example -m
```
