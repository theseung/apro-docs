# 시더 (Seeders)

## 시더 파일 위치

`htdocs/database/seeders/`

## 시더 실행

```bash
cd htdocs

# 모든 시더 실행
php artisan db:seed

# 특정 시더만 실행
php artisan db:seed --class=CompanySeeder

# 마이그레이션과 함께 실행
php artisan migrate:fresh --seed
```

## 시더 목록

| 시더 | 설명 | 실행 순서 |
|-----|------|----------|
| DatabaseSeeder | 마스터 시더 | - |
| CompanySeeder | 기업 데이터 | 1 |
| UserSeeder | 사용자 데이터 | 2 |
| PersonalityTestTypeSeeder | 검사 종류 | 3 |
| PersonalityCodeSeeder | 검사 코드 | 4 |
| PersonalityQuestionSeeder | 검사 문항 | 5 |
| PersonalitySeeder | 검사 데이터 | 6 |
| PersonalityDataSeeder | 통계 데이터 | 7 |
| PersonalityDataStatisticsFromCsvSeeder | CSV 통계 | - |

## DatabaseSeeder (마스터)

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        // 순서 중요 (외래키 의존성)
        $this->call([
            CompanySeeder::class,
            UserSeeder::class,
            PersonalityTestTypeSeeder::class,
            PersonalityCodeSeeder::class,
            PersonalityQuestionSeeder::class,
            PersonalitySeeder::class,
            PersonalityDataSeeder::class,
        ]);
    }
}
```

## CompanySeeder

```php
<?php

namespace Database\Seeders;

use App\Models\Company;
use Illuminate\Database\Seeder;

class CompanySeeder extends Seeder
{
    public function run(): void
    {
        // 슈퍼 관리자 회사 (ID=1)
        Company::create([
            'id' => 1,
            'name' => '테크솔루션',
            'type' => 'IT 서비스',
            'code' => 'TS001',
            'active' => true,
        ]);

        // 일반 회사 (ID=2)
        Company::create([
            'id' => 2,
            'name' => '디지털마케팅',
            'type' => '마케팅',
            'code' => 'DM002',
            'active' => true,
        ]);

        // 대행사 회사 (ID=3, agency_id=1)
        Company::create([
            'id' => 3,
            'name' => '테크솔루션 파트너',
            'type' => 'IT 서비스',
            'code' => 'TSP001',
            'agency_id' => 1,  // 테크솔루션이 대행
            'agency_date' => now(),
            'active' => true,
        ]);
    }
}
```

## UserSeeder

```php
<?php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        // 슈퍼 관리자
        User::create([
            'name' => '관리자',
            'email' => 'admin@example.com',
            'password' => Hash::make('password'),
            'company_id' => 1,
            'role' => 'admin',
            'level' => 3,
        ]);

        // 팀장
        User::create([
            'name' => '팀장',
            'email' => 'manager@example.com',
            'password' => Hash::make('password'),
            'company_id' => 1,
            'role' => 'manager',
            'level' => 2,
        ]);

        // 개발자 (다른 회사)
        User::create([
            'name' => '개발자',
            'email' => 'developer@example.com',
            'password' => Hash::make('password'),
            'company_id' => 2,
            'role' => 'user',
            'level' => 1,
        ]);

        // 마케터 (다른 회사)
        User::create([
            'name' => '마케터',
            'email' => 'marketer@example.com',
            'password' => Hash::make('password'),
            'company_id' => 2,
            'role' => 'user',
            'level' => 1,
        ]);
    }
}
```

## PersonalityCodeSeeder

```php
<?php

namespace Database\Seeders;

use App\Models\PersonalityCode;
use Illuminate\Database\Seeder;

class PersonalityCodeSeeder extends Seeder
{
    public function run(): void
    {
        $codes = [
            // MBTI 관련
            [
                'code' => 'EI',
                'name' => '외향-내향',
                'test_type' => 'MBTI',
                'characteristic1' => '사람들과 어울리기를 좋아함',
                'characteristic2' => '혼자 있는 시간을 선호함',
                'active' => true,
            ],
            [
                'code' => 'SN',
                'name' => '감각-직관',
                'test_type' => 'MBTI',
                'active' => true,
            ],
            [
                'code' => 'TF',
                'name' => '사고-감정',
                'test_type' => 'MBTI',
                'active' => true,
            ],
            [
                'code' => 'JP',
                'name' => '판단-인식',
                'test_type' => 'MBTI',
                'active' => true,
            ],
            // Big5 관련
            [
                'code' => 'O',
                'name' => '개방성',
                'test_type' => 'Big5',
                'active' => true,
            ],
            [
                'code' => 'C',
                'name' => '성실성',
                'test_type' => 'Big5',
                'active' => true,
            ],
            // ... 더 많은 코드
        ];

        foreach ($codes as $code) {
            PersonalityCode::create($code);
        }
    }
}
```

## PersonalityQuestionSeeder

```php
<?php

namespace Database\Seeders;

use App\Models\PersonalityQuestion;
use Illuminate\Database\Seeder;

class PersonalityQuestionSeeder extends Seeder
{
    public function run(): void
    {
        $questions = [
            [
                'code' => 'EI',
                'content' => '사람들과 함께 있을 때 에너지를 얻는다.',
                'direction' => 'positive',
                'active' => true,
            ],
            [
                'code' => 'EI',
                'content' => '혼자 있을 때 더 편안함을 느낀다.',
                'direction' => 'negative',
                'active' => true,
            ],
            // ... 수천 개의 문항
        ];

        foreach ($questions as $question) {
            PersonalityQuestion::create($question);
        }
    }
}
```

## CSV 기반 시더

대용량 데이터는 CSV 파일에서 로드합니다.

```php
<?php

namespace Database\Seeders;

use App\Models\PersonalityDataStatistics;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\File;

class PersonalityDataStatisticsFromCsvSeeder extends Seeder
{
    public function run(): void
    {
        $csvFiles = ['data1.csv', 'data2.csv', 'data3.csv'];

        foreach ($csvFiles as $file) {
            $path = base_path($file);

            if (!File::exists($path)) {
                $this->command->warn("File not found: {$file}");
                continue;
            }

            $handle = fopen($path, 'r');
            $header = fgetcsv($handle);

            while (($row = fgetcsv($handle)) !== false) {
                $data = array_combine($header, $row);

                PersonalityDataStatistics::updateOrCreate(
                    ['code' => $data['code']],
                    [
                        'name' => $data['name'],
                        'test_type' => $data['test_type'],
                        'average' => $data['average'],
                        'standard_deviation' => $data['std_dev'],
                        'top_2' => $data['top_2'] ?? null,
                        'top_5' => $data['top_5'] ?? null,
                        'top_10' => $data['top_10'] ?? null,
                        'count' => $data['count'],
                    ]
                );
            }

            fclose($handle);
        }
    }
}
```

## 시더 생성 명령어

```bash
# 새 시더 생성
php artisan make:seeder ExampleSeeder

# Factory와 함께 사용
php artisan make:factory ExampleFactory --model=Example
```

## Factory 사용 예시

```php
<?php

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    protected $model = User::class;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'password' => bcrypt('password'),
            'company_id' => 1,
            'level' => 1,
        ];
    }
}

// 시더에서 사용
User::factory()->count(10)->create();
```
