# 모델 관계

## 핵심 엔티티 관계도

```
                              ┌─────────────┐
                              │  companies  │
                              │    (1)      │
                              └──────┬──────┘
                                     │
           ┌─────────────────────────┼─────────────────────────┐
           │                         │                         │
           ▼                         ▼                         ▼
    ┌─────────────┐          ┌─────────────┐          ┌─────────────┐
    │   users     │          │ personality │          │ reputation  │
    │    (M)      │          │   _sheets   │          │   _tasks    │
    └─────────────┘          └──────┬──────┘          └──────┬──────┘
                                    │                        │
                                    ▼                        ├───────────┐
                            ┌─────────────┐                  │           │
                            │personalities│                  ▼           ▼
                            │    (M)      │          ┌───────────┐ ┌───────────┐
                            └──────┬──────┘          │ reputation│ │ reputation│
                                   │                 │_applicants│ │  _raters  │
                                   ▼                 └───────────┘ └───────────┘
                           ┌──────────────┐
                           │ personality  │
                           │ _applicants  │
                           └──────────────┘
```

## 모델별 상세

### Company

```php
class Company extends Model
{
    protected $fillable = [
        'name', 'type', 'code', 'logo', 'talent',
        'agency_id', 'agency_date', 'active'
    ];

    // 관계
    public function users() { return $this->hasMany(User::class); }
    public function agency() { return $this->belongsTo(Company::class, 'agency_id'); }
    public function agencies() { return $this->hasMany(Company::class, 'agency_id'); }
    public function personalitySheets() { return $this->hasMany(PersonalitySheet::class); }

    // 접근자
    public function agencyName() { return $this->agency?->name; }
}
```

### User

```php
class User extends Authenticatable
{
    use HasApiTokens, HasFactory;

    protected $fillable = [
        'name', 'email', 'password', 'phone', 'position',
        'company_id', 'role', 'level'
    ];

    protected $hidden = ['password', 'remember_token'];

    // 관계
    public function company() { return $this->belongsTo(Company::class); }
}
```

### Personality (인성검사)

```php
class Personality extends Model
{
    protected $primaryKey = 'idx';
    public $incrementing = false;
    protected $keyType = 'string';

    protected $fillable = [
        'idx', 'name', 'online', 'company_id', 'personality_sheet_id',
        'start_date', 'end_date', 'is_stop', 'reliability',
        's_score', 'a_score', 'b_score', 'c_score',
        'welcome_msg1', 'welcome_msg2', 'info_msg'
    ];

    protected $casts = [
        'online' => 'boolean',
        'is_stop' => 'boolean',
        'start_date' => 'datetime',
        'end_date' => 'datetime',
    ];

    // 관계
    public function company() { return $this->belongsTo(Company::class); }
    public function sheet() { return $this->belongsTo(PersonalitySheet::class, 'personality_sheet_id'); }
    public function applicants() { return $this->hasMany(PersonalityApplicant::class, 'personality_idx', 'idx'); }

    // 접근자 (appends)
    public function getStatusAttribute() {
        if ($this->is_stop) return 'stopped';
        if ($this->start_date > now()) return 'pending';
        if ($this->end_date < now()) return 'completed';
        return 'active';
    }
}
```

### PersonalityApplicant (인성검사 응시자)

```php
class PersonalityApplicant extends Model
{
    use HasApiTokens;

    protected $primaryKey = 'idx';
    public $incrementing = false;
    protected $keyType = 'string';

    protected $fillable = [
        'idx', 'personality_idx', 'company_id',
        'name', 'applicant_number', 'id', 'pw',
        'email', 'phone', 'field', 'additional_id',
        'answer', 'scoring_result', 'report_value',
        'is_complete', 'start_date', 'end_date',
        'avg_score', 'reliability_score',
        'statistics_reflected', 'is_scoring_started', 'is_scoring_completed',
        'login_count'
    ];

    protected $casts = [
        'answer' => 'array',
        'scoring_result' => 'array',
        'report_value' => 'array',
        'is_complete' => 'boolean',
        'statistics_reflected' => 'boolean',
        'is_scoring_started' => 'boolean',
        'is_scoring_completed' => 'boolean',
    ];

    // 관계
    public function personality() {
        return $this->belongsTo(Personality::class, 'personality_idx', 'idx');
    }
    public function company() { return $this->belongsTo(Company::class); }

    // 메서드
    public function startScoring() {
        $this->update(['is_scoring_started' => true]);
    }
    public function completeScoring() {
        $this->update(['is_scoring_completed' => true]);
    }
    public function resetScoring() {
        $this->update([
            'is_scoring_started' => false,
            'is_scoring_completed' => false
        ]);
    }
}
```

### PersonalitySheet (검사지)

```php
class PersonalitySheet extends Model
{
    protected $fillable = [
        'name', 'type', 'company_id', 'solve_time', 'answerType',
        'code', 'problem', 'settlement', 'problem_order',
        'display_codes', 'active'
    ];

    protected $casts = [
        'code' => 'array',
        'problem' => 'array',
        'settlement' => 'array',
        'problem_order' => 'array',
        'display_codes' => 'array',
        'active' => 'boolean',
    ];

    public function company() { return $this->belongsTo(Company::class); }
    public function personalities() { return $this->hasMany(Personality::class); }
}
```

### ReputationTask (평판진단 작업)

```php
class ReputationTask extends Model
{
    protected $primaryKey = 'idx';
    public $incrementing = false;
    protected $keyType = 'string';

    protected $fillable = [
        'idx', 'name', 'company_id', 'sheet_id',
        'rater_min', 'rater_max', 'start_date', 'end_date', 'is_stop',
        'welcome_msg1', 'welcome_msg2', 'info_msg', 'guide_msg',
        'applicant_welcome_msg1', 'applicant_welcome_msg2', 'applicant_info_msg',
        'rater_welcome_msg1', 'rater_welcome_msg2', 'rater_info_msg'
    ];

    protected $appends = ['status', 'company_name', 'sheet_name'];

    // 관계
    public function company() { return $this->belongsTo(Company::class); }
    public function sheet() { return $this->belongsTo(ReputationSheet::class, 'sheet_id'); }
    public function applicants() { return $this->hasMany(ReputationApplicant::class, 'task_idx', 'idx'); }
    public function raters() { return $this->hasMany(ReputationRater::class, 'task_idx', 'idx'); }
}
```

### ReputationRater (평가자)

```php
class ReputationRater extends Model
{
    use HasApiTokens;

    protected $primaryKey = 'idx';
    public $incrementing = false;
    protected $keyType = 'string';

    protected $fillable = [
        'idx', 'task_idx', 'applicant_idx', 'company_id',
        'name', 'email', 'company', 'position', 'relationship',
        'exchange', 'is_complete', 'answer', 'complete_date',
        'email_sent_at', 'scoring_record',
        'statistics_updated', 'statistics_updated_at'
    ];

    protected $casts = [
        'answer' => 'array',
        'is_complete' => 'boolean',
        'exchange' => 'boolean',
        'statistics_updated' => 'boolean',
    ];

    // 관계
    public function task() { return $this->belongsTo(ReputationTask::class, 'task_idx', 'idx'); }
    public function applicant() { return $this->belongsTo(ReputationApplicant::class, 'applicant_idx', 'idx'); }
    public function company() { return $this->belongsTo(Company::class); }
}
```

### ReportAccessToken (보고서 접근 토큰)

```php
class ReportAccessToken extends Model
{
    protected $fillable = [
        'token', 'company_id', 'personality_idx', 'applicant_idx',
        'blind_name_fg', 'blind_field_fg', 'blind_identifier_fg', 'blind_exam_number_fg',
        'population_source_fg', 'field_rank_fg',
        'expires_at', 'used_at'
    ];

    // 토큰 생성
    public static function generateToken(): string {
        return bin2hex(random_bytes(32)); // 64자
    }

    public static function createToken(array $data): self {
        return self::create([
            'token' => self::generateToken(),
            ...$data,
            'expires_at' => now()->addHours(24)
        ]);
    }

    // 검증
    public static function verifyToken(string $token): ?self {
        return self::where('token', $token)
            ->where('expires_at', '>', now())
            ->whereNull('used_at')
            ->first();
    }

    public function markAsUsed(): void {
        $this->update(['used_at' => now()]);
    }
}
```

## 관계 요약표

| 부모 | 자식 | 관계 | 외래키 |
|-----|-----|-----|--------|
| Company | User | 1:N | company_id |
| Company | PersonalitySheet | 1:N | company_id |
| Company | Company (대행) | 1:N | agency_id |
| PersonalitySheet | Personality | 1:N | personality_sheet_id |
| Personality | PersonalityApplicant | 1:N | personality_idx (UUID) |
| ReputationTask | ReputationApplicant | 1:N | task_idx (UUID) |
| ReputationTask | ReputationRater | 1:N | task_idx (UUID) |
| ReputationApplicant | ReputationRater | 1:N | applicant_idx (UUID) |
| MultifacetedTask | MultifacetedApplicant | 1:N | task_idx (UUID) |
| MultifacetedTask | MultifacetedRater | 1:N | task_idx (UUID) |
