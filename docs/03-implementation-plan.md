# TimeSeriesViewer SaaS - Detailed Implementation Plan

## 1. Executive Summary

This document provides a comprehensive implementation plan for building the TimeSeriesViewer SaaS platform. The plan is organized into phases, with each phase building upon the previous one to deliver incremental value while managing risk.

### 1.1 Project Timeline Overview

| Phase | Duration | Focus |
|-------|----------|-------|
| Phase 0: Foundation | 2-3 weeks | Setup, infrastructure, CI/CD |
| Phase 1: Core MVP | 8-10 weeks | Essential features |
| Phase 2: Enhanced Features | 6-8 weeks | Advanced capabilities |
| Phase 3: Production Readiness | 4-6 weeks | Scaling, security, monitoring |
| Phase 4: Launch | 2-3 weeks | Beta testing, documentation |
| **Total** | **22-30 weeks** | |

---

## 2. Phase 0: Foundation Setup

**Duration:** 2-3 weeks

### 2.1 Development Environment Setup

#### 2.1.1 Repository Structure

```
TimeSeriesViewer/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   ├── cd-staging.yml
│   │   └── cd-production.yml
│   └── CODEOWNERS
│
├── docs/
│   ├── architecture/
│   ├── api/
│   └── runbooks/
│
├── infrastructure/
│   ├── terraform/
│   │   ├── environments/
│   │   │   ├── dev/
│   │   │   ├── staging/
│   │   │   └── production/
│   │   ├── modules/
│   │   │   ├── networking/
│   │   │   ├── compute/
│   │   │   ├── data/
│   │   │   └── security/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── scripts/
│
├── src/
│   ├── backend/
│   │   ├── TimeSeriesViewer.Api/
│   │   ├── TimeSeriesViewer.Domain/
│   │   ├── TimeSeriesViewer.Infrastructure/
│   │   ├── TimeSeriesViewer.Functions/
│   │   └── TimeSeriesViewer.Tests/
│   │
│   └── frontend/
│       ├── public/
│       ├── src/
│       │   ├── components/
│       │   ├── pages/
│       │   ├── hooks/
│       │   ├── services/
│       │   ├── store/
│       │   ├── types/
│       │   └── utils/
│       ├── package.json
│       ├── tsconfig.json
│       └── vite.config.ts
│
├── docker/
│   ├── Dockerfile.api
│   ├── Dockerfile.functions
│   └── docker-compose.yml
│
├── .editorconfig
├── .gitignore
└── README.md
```

#### 2.1.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Create GitHub repository with branch protection | High | 2h | DevOps |
| Set up repository structure | High | 4h | Tech Lead |
| Configure .editorconfig and linting rules | Medium | 2h | Tech Lead |
| Create development environment documentation | Medium | 4h | Tech Lead |
| Set up Docker Compose for local development | High | 8h | DevOps |

### 2.2 Azure Infrastructure Foundation

#### 2.2.1 Initial Azure Resources

```hcl
# Core resource groups
resource "azurerm_resource_group" "main" {
  name     = "rg-tsv-${var.environment}"
  location = var.location
  tags     = var.common_tags
}

# Key Vault (first to store other secrets)
resource "azurerm_key_vault" "main" {
  name                = "kv-tsv-${var.environment}"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  tenant_id           = data.azurerm_client_config.current.tenant_id
  sku_name           = "standard"
}

# Container Registry
resource "azurerm_container_registry" "main" {
  name                = "acrtsv${var.environment}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  sku                 = "Standard"
  admin_enabled       = false
}

# Storage Account
resource "azurerm_storage_account" "main" {
  name                     = "sttsv${var.environment}"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

#### 2.2.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Create Azure subscription and management groups | High | 4h | DevOps |
| Set up Terraform state backend | High | 2h | DevOps |
| Create base infrastructure modules | High | 16h | DevOps |
| Deploy development environment | High | 4h | DevOps |
| Configure Azure AD B2C tenant | High | 8h | DevOps |
| Set up initial Azure Key Vault | High | 2h | DevOps |

### 2.3 CI/CD Pipeline Setup

#### 2.3.1 GitHub Actions Workflows

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  backend-build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - name: Restore dependencies
        run: dotnet restore src/backend/TimeSeriesViewer.sln
      - name: Build
        run: dotnet build src/backend/TimeSeriesViewer.sln --no-restore
      - name: Test
        run: dotnet test src/backend/TimeSeriesViewer.sln --no-build --verbosity normal

  frontend-build:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: src/frontend
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: src/frontend/package-lock.json
      - name: Install dependencies
        run: npm ci
      - name: Lint
        run: npm run lint
      - name: Test
        run: npm run test
      - name: Build
        run: npm run build

  docker-build:
    needs: [backend-build, frontend-build]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker images
        run: |
          docker build -f docker/Dockerfile.api -t tsv-api:${{ github.sha }} .
          docker build -f docker/Dockerfile.functions -t tsv-functions:${{ github.sha }} .
```

#### 2.3.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Create CI pipeline for backend | High | 4h | DevOps |
| Create CI pipeline for frontend | High | 4h | DevOps |
| Set up Docker build pipeline | High | 4h | DevOps |
| Configure CD pipeline for staging | High | 8h | DevOps |
| Set up infrastructure deployment pipeline | High | 8h | DevOps |
| Configure secrets management in GitHub | Medium | 2h | DevOps |

### 2.4 Phase 0 Deliverables

- [ ] Development environment fully operational
- [ ] Azure infrastructure foundation deployed
- [ ] CI/CD pipelines functional
- [ ] Team onboarded and productive

---

## 3. Phase 1: Core MVP

**Duration:** 8-10 weeks

### 3.1 Sprint 1: Backend Foundation (Weeks 1-2)

#### 3.1.1 Project Setup

```csharp
// Solution structure
TimeSeriesViewer.sln
├── src/
│   ├── TimeSeriesViewer.Api/           // ASP.NET Core Web API
│   ├── TimeSeriesViewer.Domain/        // Domain models, interfaces
│   ├── TimeSeriesViewer.Application/   // Application services, DTOs
│   └── TimeSeriesViewer.Infrastructure/ // Data access, external services
├── tests/
│   ├── TimeSeriesViewer.Tests.Unit/
│   └── TimeSeriesViewer.Tests.Integration/
└── tools/
    └── TimeSeriesViewer.DbMigrations/

// Key NuGet packages
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="8.0.*" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.*" />
<PackageReference Include="Azure.Storage.Blobs" Version="12.*" />
<PackageReference Include="Azure.Identity" Version="1.*" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="6.*" />
<PackageReference Include="Serilog.AspNetCore" Version="8.*" />
<PackageReference Include="MediatR" Version="12.*" />
<PackageReference Include="FluentValidation" Version="11.*" />
```

#### 3.1.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Create .NET solution and projects | High | 4h | Backend Dev |
| Configure dependency injection | High | 4h | Backend Dev |
| Set up Entity Framework Core | High | 8h | Backend Dev |
| Implement base repository pattern | High | 8h | Backend Dev |
| Configure Serilog logging | Medium | 4h | Backend Dev |
| Set up health check endpoints | Medium | 2h | Backend Dev |
| Configure Swagger/OpenAPI | Medium | 4h | Backend Dev |
| Create initial database migrations | High | 8h | Backend Dev |

### 3.2 Sprint 2: Authentication & Users (Weeks 3-4)

#### 3.2.1 Azure AD B2C Configuration

```xml
<!-- User flows to configure -->
- Sign up / Sign in (B2C_1_SignUpSignIn)
- Password reset (B2C_1_PasswordReset)
- Profile edit (B2C_1_ProfileEdit)

<!-- Identity providers -->
- Local accounts (email)
- Google
- Microsoft
- Apple (requires Apple Developer account)
```

#### 3.2.2 User Service Implementation

```csharp
// User endpoints
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class UsersController : ControllerBase
{
    [HttpGet("me")]
    public async Task<ActionResult<UserDto>> GetCurrentUser()

    [HttpPut("me")]
    public async Task<ActionResult<UserDto>> UpdateProfile(UpdateUserRequest request)

    [HttpGet("me/preferences")]
    public async Task<ActionResult<UserPreferencesDto>> GetPreferences()

    [HttpPut("me/preferences")]
    public async Task<ActionResult> UpdatePreferences(UserPreferencesDto preferences)

    [HttpDelete("me")]
    public async Task<ActionResult> DeleteAccount()
}
```

#### 3.2.3 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Configure Azure AD B2C tenant | High | 16h | DevOps |
| Set up identity providers (Google, Microsoft, Apple) | High | 16h | DevOps |
| Implement JWT authentication middleware | High | 8h | Backend Dev |
| Create User domain model | High | 4h | Backend Dev |
| Implement User repository | High | 8h | Backend Dev |
| Create User Service with CRUD operations | High | 16h | Backend Dev |
| Implement user preferences | Medium | 8h | Backend Dev |
| Write unit tests for User service | High | 8h | Backend Dev |
| Write integration tests for auth flow | High | 8h | Backend Dev |

### 3.3 Sprint 3: File Service (Weeks 5-6)

#### 3.3.1 File Upload Flow

```csharp
// File upload endpoints
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class FilesController : ControllerBase
{
    // Initialize upload - returns SAS URL for direct upload to Blob
    [HttpPost("upload/init")]
    public async Task<ActionResult<FileUploadInitResponse>> InitializeUpload(
        InitializeUploadRequest request)

    // Complete upload - triggers processing
    [HttpPost("upload/{fileId}/complete")]
    public async Task<ActionResult<FileDto>> CompleteUpload(Guid fileId)

    // Chunked upload for large files
    [HttpPost("upload/{fileId}/chunk")]
    public async Task<ActionResult> UploadChunk(
        Guid fileId,
        [FromQuery] int chunkIndex,
        IFormFile chunk)

    [HttpGet]
    public async Task<ActionResult<PagedResult<FileDto>>> GetFiles(
        [FromQuery] FileQueryParams queryParams)

    [HttpGet("{id}")]
    public async Task<ActionResult<FileDto>> GetFile(Guid id)

    [HttpDelete("{id}")]
    public async Task<ActionResult> DeleteFile(Guid id)

    [HttpGet("{id}/columns")]
    public async Task<ActionResult<List<FileColumnDto>>> GetColumns(Guid id)

    [HttpGet("{id}/data")]
    public async Task<ActionResult<TimeSeriesDataDto>> GetData(
        Guid id,
        [FromQuery] DataQueryParams queryParams)
}
```

#### 3.3.2 File Processing Azure Function

```csharp
public class FileProcessorFunction
{
    [FunctionName("ProcessUploadedFile")]
    public async Task Run(
        [BlobTrigger("raw/{userId}/{fileId}/{fileName}", 
            Connection = "StorageConnection")] Stream blob,
        string userId,
        string fileId,
        string fileName,
        ILogger log)
    {
        // 1. Detect file format
        // 2. Parse file (CSV, Parquet)
        // 3. Extract schema/columns
        // 4. Generate metadata
        // 5. Store processed data
        // 6. Update file status in database
        // 7. Send SignalR notification
    }
}
```

#### 3.3.3 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Create File domain model | High | 4h | Backend Dev |
| Implement file repository | High | 8h | Backend Dev |
| Create Azure Blob Storage service | High | 16h | Backend Dev |
| Implement SAS URL generation | High | 8h | Backend Dev |
| Create File Processing Azure Function | High | 24h | Backend Dev |
| Implement CSV parser | High | 16h | Backend Dev |
| Implement Parquet parser | High | 16h | Backend Dev |
| Add file size/type validation | High | 4h | Backend Dev |
| Implement chunked upload | Medium | 16h | Backend Dev |
| Set up SignalR for upload notifications | Medium | 8h | Backend Dev |
| Write unit tests for file service | High | 8h | Backend Dev |
| Write integration tests for file upload | High | 12h | Backend Dev |

### 3.4 Sprint 4: Frontend Foundation (Weeks 5-6, parallel)

#### 3.4.1 Frontend Setup

```bash
# Create React project with Vite
npm create vite@latest frontend -- --template react-ts

# Install core dependencies
npm install @tanstack/react-query axios react-router-dom zustand
npm install @mui/material @mui/icons-material @emotion/react @emotion/styled
npm install echarts echarts-for-react
npm install react-dropzone
npm install @azure/msal-browser @azure/msal-react

# Install dev dependencies
npm install -D @types/node vitest @testing-library/react
npm install -D eslint prettier eslint-config-prettier
npm install -D msw  # Mock Service Worker for testing
```

#### 3.4.2 Component Structure

```typescript
// src/App.tsx
import { MsalProvider } from "@azure/msal-react";
import { QueryClientProvider } from "@tanstack/react-query";
import { RouterProvider } from "react-router-dom";

function App() {
  return (
    <MsalProvider instance={msalInstance}>
      <QueryClientProvider client={queryClient}>
        <RouterProvider router={router} />
      </QueryClientProvider>
    </MsalProvider>
  );
}

// src/router.tsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <PublicLayout />,
    children: [
      { index: true, element: <LandingPage /> },
      { path: "login", element: <LoginPage /> },
    ],
  },
  {
    path: "/app",
    element: <ProtectedLayout />,
    children: [
      { index: true, element: <DashboardPage /> },
      { path: "projects/:id", element: <ProjectPage /> },
      { path: "files", element: <FilesPage /> },
      { path: "settings", element: <SettingsPage /> },
    ],
  },
]);
```

#### 3.4.3 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Create React project with Vite | High | 2h | Frontend Dev |
| Configure TypeScript and ESLint | High | 4h | Frontend Dev |
| Set up routing structure | High | 4h | Frontend Dev |
| Implement MSAL authentication | High | 16h | Frontend Dev |
| Create layout components | High | 8h | Frontend Dev |
| Implement file upload component | High | 16h | Frontend Dev |
| Create file list component | High | 8h | Frontend Dev |
| Set up API client service | High | 8h | Frontend Dev |
| Configure state management | High | 8h | Frontend Dev |
| Create basic UI components | Medium | 16h | Frontend Dev |

### 3.5 Sprint 5: Chart Visualization (Weeks 7-8)

#### 3.5.1 Chart Service Backend

```csharp
[ApiController]
[Route("api/projects/{projectId}/charts")]
[Authorize]
public class ChartsController : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<ChartDto>> CreateChart(
        Guid projectId,
        CreateChartRequest request)

    [HttpGet]
    public async Task<ActionResult<List<ChartDto>>> GetCharts(Guid projectId)

    [HttpGet("{chartId}")]
    public async Task<ActionResult<ChartDto>> GetChart(
        Guid projectId,
        Guid chartId)

    [HttpPut("{chartId}")]
    public async Task<ActionResult<ChartDto>> UpdateChart(
        Guid projectId,
        Guid chartId,
        UpdateChartRequest request)

    [HttpDelete("{chartId}")]
    public async Task<ActionResult> DeleteChart(
        Guid projectId,
        Guid chartId)

    [HttpGet("{chartId}/data")]
    public async Task<ActionResult<ChartDataResponse>> GetChartData(
        Guid projectId,
        Guid chartId,
        [FromQuery] ChartDataQueryParams queryParams)
}

// Chart data endpoint returns aggregated/sampled data for visualization
public class ChartDataResponse
{
    public List<SeriesData> Series { get; set; }
    public TimeRange TimeRange { get; set; }
    public int TotalPoints { get; set; }
    public int SampledPoints { get; set; }
}
```

#### 3.5.2 Chart Component

```typescript
// src/components/Chart/Chart.tsx
import ReactECharts from 'echarts-for-react';

interface ChartProps {
  chartConfig: ChartConfig;
  data: ChartData;
  onZoom?: (range: TimeRange) => void;
  onSeriesClick?: (series: SeriesInfo) => void;
}

export const Chart: React.FC<ChartProps> = ({
  chartConfig,
  data,
  onZoom,
  onSeriesClick,
}) => {
  const options = useMemo(() => buildChartOptions(chartConfig, data), [chartConfig, data]);

  return (
    <div className="chart-container">
      <ChartToolbar config={chartConfig} />
      <ReactECharts
        option={options}
        style={{ height: '400px' }}
        onEvents={{
          datazoom: (params) => onZoom?.(extractRange(params)),
          click: (params) => onSeriesClick?.(extractSeriesInfo(params)),
        }}
      />
    </div>
  );
};
```

#### 3.5.3 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Create Chart domain model | High | 4h | Backend Dev |
| Implement chart repository | High | 8h | Backend Dev |
| Create chart service | High | 16h | Backend Dev |
| Implement data aggregation/sampling | High | 16h | Backend Dev |
| Create Chart React component | High | 24h | Frontend Dev |
| Implement chart configuration panel | High | 16h | Frontend Dev |
| Add series color/style picker | Medium | 8h | Frontend Dev |
| Implement zoom/pan functionality | High | 8h | Frontend Dev |
| Add tooltip customization | Medium | 4h | Frontend Dev |
| Create chart grid layout | Medium | 8h | Frontend Dev |

### 3.6 Sprint 6: Projects & Integration (Weeks 9-10)

#### 3.6.1 Project Service

```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class ProjectsController : ControllerBase
{
    [HttpPost]
    public async Task<ActionResult<ProjectDto>> CreateProject(CreateProjectRequest request)

    [HttpGet]
    public async Task<ActionResult<PagedResult<ProjectDto>>> GetProjects(
        [FromQuery] ProjectQueryParams queryParams)

    [HttpGet("{id}")]
    public async Task<ActionResult<ProjectDetailDto>> GetProject(Guid id)

    [HttpPut("{id}")]
    public async Task<ActionResult<ProjectDto>> UpdateProject(
        Guid id,
        UpdateProjectRequest request)

    [HttpDelete("{id}")]
    public async Task<ActionResult> DeleteProject(Guid id)

    [HttpPost("{id}/files")]
    public async Task<ActionResult> AddFileToProject(Guid id, AddFileRequest request)

    [HttpDelete("{id}/files/{fileId}")]
    public async Task<ActionResult> RemoveFileFromProject(Guid id, Guid fileId)
}
```

#### 3.6.2 Project Dashboard

```typescript
// src/pages/ProjectPage/ProjectPage.tsx
export const ProjectPage: React.FC = () => {
  const { projectId } = useParams<{ projectId: string }>();
  const { data: project, isLoading } = useProject(projectId);

  return (
    <div className="project-page">
      <ProjectHeader project={project} />

      <div className="project-content">
        <div className="charts-panel">
          <ChartGrid projectId={projectId} />
        </div>

        <div className="sidebar-panel">
          <FilePanel projectId={projectId} />
          <ColumnSelector projectId={projectId} />
        </div>
      </div>
    </div>
  );
};
```

#### 3.6.3 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Create Project domain model | High | 4h | Backend Dev |
| Implement project repository | High | 8h | Backend Dev |
| Create project service | High | 16h | Backend Dev |
| Implement file-project association | High | 8h | Backend Dev |
| Create project dashboard page | High | 24h | Frontend Dev |
| Implement project list/grid | High | 8h | Frontend Dev |
| Create project creation flow | High | 8h | Frontend Dev |
| Implement column selector component | High | 16h | Frontend Dev |
| Add drag-drop for chart series | Medium | 8h | Frontend Dev |
| End-to-end integration testing | High | 16h | QA |

### 3.7 Phase 1 Deliverables

- [ ] User authentication with social logins working
- [ ] File upload and parsing (CSV, Parquet) functional
- [ ] Basic chart visualization operational
- [ ] Project management working
- [ ] Column selection and chart configuration
- [ ] All unit and integration tests passing
- [ ] API documentation complete

---

## 4. Phase 2: Enhanced Features

**Duration:** 6-8 weeks

### 4.1 Sprint 7: Time Axis Normalization (Weeks 1-2)

#### 4.1.1 Normalization Logic

```csharp
public class TimeAxisNormalizer
{
    /// <summary>
    /// Normalizes multiple time series to a common time axis.
    /// Handles different sampling rates and start times.
    /// </summary>
    public NormalizedDataSet Normalize(
        IEnumerable<TimeSeries> series,
        NormalizationOptions options)
    {
        // 1. Determine common time range
        var commonRange = CalculateCommonTimeRange(series, options.RangeMode);

        // 2. Determine target sampling rate
        var targetRate = DetermineTargetSamplingRate(series, options.SamplingMode);

        // 3. Generate common time axis
        var timeAxis = GenerateTimeAxis(commonRange, targetRate);

        // 4. Interpolate/resample each series to common axis
        var normalizedSeries = series.Select(s =>
            InterpolateToAxis(s, timeAxis, options.InterpolationMethod));

        return new NormalizedDataSet
        {
            TimeAxis = timeAxis,
            Series = normalizedSeries.ToList()
        };
    }
}
```

#### 4.1.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Design normalization algorithm | High | 8h | Backend Dev |
| Implement time range calculation | High | 8h | Backend Dev |
| Implement interpolation methods | High | 16h | Backend Dev |
| Create normalization service | High | 16h | Backend Dev |
| Add normalization UI controls | High | 16h | Frontend Dev |
| Implement visual time axis sync | High | 12h | Frontend Dev |
| Write unit tests | High | 8h | Backend Dev |
| Performance optimization | Medium | 8h | Backend Dev |

### 4.2 Sprint 8: Python Transformations (Weeks 3-4)

#### 4.2.1 Expression Engine

```csharp
public class PythonExpressionEngine
{
    private readonly IScriptEngine _engine;

    public PythonExpressionEngine()
    {
        // Use IronPython or Python.NET
        _engine = CreateSandboxedEngine();
    }

    public ValidationResult Validate(string expression)
    {
        // Parse and validate syntax
        // Check for forbidden operations
        // Verify variable references
    }

    public TransformResult Execute(
        string expression,
        Dictionary<string, double[]> variables)
    {
        // Execute in sandbox
        // Apply timeout
        // Return transformed data
    }

    // Supported operations:
    // - Basic math: +, -, *, /, **, %
    // - Functions: sin, cos, tan, log, exp, sqrt, abs, etc.
    // - Aggregations: mean, std, min, max, sum
    // - Array operations: cumsum, diff, shift
    // - Conditionals: where, clip
}
```

#### 4.2.2 Transformation UI

```typescript
// src/components/TransformationEditor/TransformationEditor.tsx
export const TransformationEditor: React.FC<{
  series: ChartSeries;
  onApply: (expression: string) => void;
}> = ({ series, onApply }) => {
  const [expression, setExpression] = useState('');
  const [preview, setPreview] = useState<PreviewData | null>(null);
  const [error, setError] = useState<string | null>(null);

  const validateExpression = async () => {
    const result = await api.transformations.validate(expression);
    setError(result.isValid ? null : result.errorMessage);
  };

  const previewTransformation = async () => {
    const data = await api.transformations.preview({
      expression,
      seriesId: series.id,
      sampleSize: 100,
    });
    setPreview(data);
  };

  return (
    <div className="transformation-editor">
      <CodeEditor
        value={expression}
        onChange={setExpression}
        language="python"
        onBlur={validateExpression}
      />
      {error && <Alert severity="error">{error}</Alert>}
      <Button onClick={previewTransformation}>Preview</Button>
      {preview && <MiniChart data={preview} />}
      <Button onClick={() => onApply(expression)} disabled={!!error}>
        Apply
      </Button>
    </div>
  );
};
```

#### 4.2.3 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Research and select Python engine | High | 8h | Backend Dev |
| Implement sandboxed execution | High | 24h | Backend Dev |
| Create expression validator | High | 16h | Backend Dev |
| Implement common math functions | High | 16h | Backend Dev |
| Create transformation API endpoints | High | 8h | Backend Dev |
| Build expression editor component | High | 16h | Frontend Dev |
| Implement preview functionality | High | 12h | Frontend Dev |
| Add transformation history | Medium | 8h | Frontend Dev |
| Security review and testing | High | 16h | Security |

### 4.3 Sprint 9: Project Versioning (Weeks 5-6)

#### 4.3.1 Versioning Service

```csharp
public class ProjectVersioningService
{
    public async Task<ProjectVersion> CreateVersion(
        Guid projectId,
        string name,
        string description)
    {
        var project = await _projectRepository.GetWithDetails(projectId);

        var snapshot = new ProjectSnapshot
        {
            Charts = project.Charts.Select(c => SerializeChart(c)).ToList(),
            Files = project.Files.Select(f => f.Id).ToList(),
            Transformations = project.Transformations.ToList(),
        };

        var version = new ProjectVersion
        {
            ProjectId = projectId,
            VersionNumber = await GetNextVersionNumber(projectId),
            Name = name,
            Description = description,
            Configuration = JsonSerializer.Serialize(snapshot),
        };

        return await _versionRepository.Create(version);
    }

    public async Task RestoreVersion(Guid projectId, int versionNumber)
    {
        var version = await _versionRepository.Get(projectId, versionNumber);
        var snapshot = JsonSerializer.Deserialize<ProjectSnapshot>(version.Configuration);

        await _projectService.RestoreFromSnapshot(projectId, snapshot);
    }
}
```

#### 4.3.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Design version snapshot format | High | 4h | Backend Dev |
| Implement version service | High | 16h | Backend Dev |
| Create version comparison logic | Medium | 8h | Backend Dev |
| Add version API endpoints | High | 8h | Backend Dev |
| Create version history UI | High | 16h | Frontend Dev |
| Implement version restore flow | High | 12h | Frontend Dev |
| Add version comparison view | Medium | 8h | Frontend Dev |
| Write integration tests | High | 8h | QA |

### 4.4 Sprint 10: Export Functionality (Weeks 7-8)

#### 4.4.1 Export Service

```csharp
public class ExportService
{
    public async Task<ExportJob> CreateExport(ExportRequest request)
    {
        var job = new ExportJob
        {
            Id = Guid.NewGuid(),
            ProjectId = request.ProjectId,
            ChartId = request.ChartId,
            Format = request.Format,
            Status = ExportStatus.Pending,
        };

        await _exportRepository.Create(job);
        await _queue.EnqueueAsync(new ExportMessage { JobId = job.Id });

        return job;
    }
}

// Azure Function for export generation
public class ExportGeneratorFunction
{
    [FunctionName("GenerateExport")]
    public async Task Run(
        [QueueTrigger("exports")] ExportMessage message,
        ILogger log)
    {
        var job = await _exportRepository.Get(message.JobId);

        try
        {
            var chartData = await _chartService.GetChartData(job.ChartId);
            var image = await RenderChart(chartData, job.Format);

            var blobPath = await _blobStorage.Upload(image, job.Id, job.Format);

            job.Status = ExportStatus.Completed;
            job.BlobPath = blobPath;
            job.ExpiresAt = DateTime.UtcNow.AddHours(24);
        }
        catch (Exception ex)
        {
            job.Status = ExportStatus.Failed;
            job.ErrorMessage = ex.Message;
        }

        await _exportRepository.Update(job);
    }
}
```

#### 4.4.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Design export job workflow | High | 4h | Backend Dev |
| Implement PNG export | High | 16h | Backend Dev |
| Implement PDF export | High | 16h | Backend Dev |
| Implement SVG export | Medium | 8h | Backend Dev |
| Create export Azure Function | High | 16h | Backend Dev |
| Add export UI | High | 12h | Frontend Dev |
| Implement download flow | High | 8h | Frontend Dev |
| Add batch export | Medium | 8h | Backend Dev |

### 4.5 Phase 2 Deliverables

- [ ] Time axis normalization working across multiple files
- [ ] Python expression evaluation functional
- [ ] Project versioning with restore capability
- [ ] Chart export in PNG, PDF, SVG formats
- [ ] All new features fully tested
- [ ] Updated documentation

---

## 5. Phase 3: Production Readiness

**Duration:** 4-6 weeks

### 5.1 Sprint 11: Payment Integration (Weeks 1-2)

#### 5.1.1 Stripe Integration

```csharp
public class StripeService
{
    public async Task<string> CreateCustomer(User user)
    {
        var options = new CustomerCreateOptions
        {
            Email = user.Email,
            Name = user.Name,
            Metadata = new Dictionary<string, string>
            {
                { "userId", user.Id.ToString() }
            }
        };

        var customer = await _customerService.CreateAsync(options);
        return customer.Id;
    }

    public async Task<string> CreateCheckoutSession(
        string customerId,
        string priceId,
        string successUrl,
        string cancelUrl)
    {
        var options = new SessionCreateOptions
        {
            Customer = customerId,
            PaymentMethodTypes = new List<string> { "card" },
            LineItems = new List<SessionLineItemOptions>
            {
                new SessionLineItemOptions
                {
                    Price = priceId,
                    Quantity = 1,
                }
            },
            Mode = "subscription",
            SuccessUrl = successUrl,
            CancelUrl = cancelUrl,
        };

        var session = await _sessionService.CreateAsync(options);
        return session.Url;
    }

    public async Task HandleWebhook(string payload, string signature)
    {
        var stripeEvent = EventUtility.ConstructEvent(
            payload,
            signature,
            _webhookSecret
        );

        switch (stripeEvent.Type)
        {
            case Events.CustomerSubscriptionCreated:
                await HandleSubscriptionCreated(stripeEvent);
                break;
            case Events.CustomerSubscriptionUpdated:
                await HandleSubscriptionUpdated(stripeEvent);
                break;
            case Events.CustomerSubscriptionDeleted:
                await HandleSubscriptionDeleted(stripeEvent);
                break;
            case Events.InvoicePaymentFailed:
                await HandlePaymentFailed(stripeEvent);
                break;
        }
    }
}
```

#### 5.1.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Set up Stripe account and products | High | 4h | Business |
| Implement Stripe customer management | High | 8h | Backend Dev |
| Create checkout flow | High | 16h | Backend Dev |
| Implement webhook handlers | High | 16h | Backend Dev |
| Create subscription management UI | High | 16h | Frontend Dev |
| Implement upgrade/downgrade flow | High | 12h | Full Stack |
| Add usage limit enforcement | High | 16h | Backend Dev |
| Create billing dashboard | Medium | 8h | Frontend Dev |

### 5.2 Sprint 12: Security & Compliance (Weeks 3-4)

#### 5.2.1 Security Enhancements

```csharp
// Rate limiting configuration
services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("api", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 100;
        opt.QueueLimit = 10;
    });

    options.AddFixedWindowLimiter("upload", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 10;
    });
});

// Input validation
public class FileUploadRequestValidator : AbstractValidator<FileUploadRequest>
{
    public FileUploadRequestValidator()
    {
        RuleFor(x => x.FileName)
            .NotEmpty()
            .MaximumLength(255)
            .Must(BeValidFileName).WithMessage("Invalid file name");

        RuleFor(x => x.ContentType)
            .NotEmpty()
            .Must(BeSupportedContentType).WithMessage("Unsupported file type");

        RuleFor(x => x.Size)
            .GreaterThan(0)
            .LessThanOrEqualTo(1_073_741_824); // 1GB max
    }
}
```

#### 5.2.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Security audit and penetration testing | High | 40h | Security |
| Implement rate limiting | High | 8h | Backend Dev |
| Add input validation | High | 16h | Backend Dev |
| Implement GDPR compliance (data export, deletion) | High | 24h | Backend Dev |
| Set up Azure DDoS protection | High | 4h | DevOps |
| Configure WAF rules | High | 8h | DevOps |
| Implement audit logging | High | 16h | Backend Dev |
| Security documentation | Medium | 8h | Tech Lead |

### 5.3 Sprint 13: Monitoring & Performance (Weeks 5-6)

#### 5.3.1 Monitoring Setup

```csharp
// Application Insights configuration
services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = configuration["ApplicationInsights:ConnectionString"];
});

// Custom telemetry
public class MetricsService
{
    private readonly TelemetryClient _telemetry;

    public void TrackFileUpload(Guid fileId, long size, TimeSpan duration)
    {
        _telemetry.TrackEvent("FileUploaded", new Dictionary<string, string>
        {
            { "FileId", fileId.ToString() }
        }, new Dictionary<string, double>
        {
            { "FileSize", size },
            { "DurationMs", duration.TotalMilliseconds }
        });
    }

    public void TrackChartRender(Guid chartId, int dataPoints, TimeSpan duration)
    {
        _telemetry.TrackMetric("ChartRenderDuration", duration.TotalMilliseconds,
            new Dictionary<string, string>
            {
                { "ChartId", chartId.ToString() },
                { "DataPoints", dataPoints.ToString() }
            });
    }
}
```

#### 5.3.2 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Configure Application Insights | High | 8h | DevOps |
| Set up custom metrics | High | 16h | Backend Dev |
| Create monitoring dashboards | High | 16h | DevOps |
| Configure alerting rules | High | 8h | DevOps |
| Performance testing and optimization | High | 24h | Backend Dev |
| Database query optimization | High | 16h | Backend Dev |
| Implement caching strategy | High | 16h | Backend Dev |
| Create runbooks for common issues | Medium | 8h | DevOps |

### 5.4 Phase 3 Deliverables

- [ ] Payment system fully operational
- [ ] Security audit passed
- [ ] GDPR compliance implemented
- [ ] Monitoring and alerting configured
- [ ] Performance targets met
- [ ] Runbooks documented

---

## 6. Phase 4: Launch

**Duration:** 2-3 weeks

### 6.1 Sprint 14: Beta & Documentation (Weeks 1-2)

#### 6.1.1 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Recruit beta testers | High | 8h | Product |
| Deploy to production environment | High | 8h | DevOps |
| Conduct beta testing | High | 40h | QA |
| Address beta feedback | High | 40h | Dev Team |
| Create user documentation | High | 24h | Tech Writer |
| Create video tutorials | Medium | 16h | Marketing |
| Set up support system | High | 8h | Support |
| Final security review | High | 8h | Security |

### 6.2 Sprint 15: Launch (Week 3)

#### 6.2.1 Tasks

| Task | Priority | Effort | Owner |
|------|----------|--------|-------|
| Production deployment verification | High | 8h | DevOps |
| Launch announcement | High | 4h | Marketing |
| Monitor launch metrics | High | 16h | All |
| Handle initial support tickets | High | 16h | Support |
| Post-launch bug fixes | High | Variable | Dev Team |

### 6.3 Phase 4 Deliverables

- [ ] Beta testing complete
- [ ] All critical issues resolved
- [ ] Documentation published
- [ ] Production deployed and stable
- [ ] Support channels operational
- [ ] Launch complete

---

## 7. Resource Requirements

### 7.1 Team Structure

| Role | Count | Phase 0-1 | Phase 2-3 | Phase 4 |
|------|-------|-----------|-----------|---------|
| Tech Lead | 1 | Full-time | Full-time | Full-time |
| Backend Developer | 2 | Full-time | Full-time | Full-time |
| Frontend Developer | 2 | Full-time | Full-time | Full-time |
| DevOps Engineer | 1 | Full-time | Full-time | Part-time |
| QA Engineer | 1 | Part-time | Full-time | Full-time |
| Security Specialist | 0.5 | Part-time | Part-time | Part-time |
| Product Manager | 1 | Part-time | Full-time | Full-time |
| UX Designer | 0.5 | Part-time | Part-time | Part-time |

### 7.2 Estimated Costs (Monthly)

| Category | Phase 1-2 (Dev) | Phase 3-4 (Production) |
|----------|-----------------|------------------------|
| Azure Infrastructure | €500-1,000 | €2,000-5,000 |
| Development Tools | €200 | €200 |
| Third-party Services | €200 | €500 |
| **Total** | **€900-1,400** | **€2,700-5,700** |

---

## 8. Risk Management

### 8.1 Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Large file processing performance | Medium | High | Stream processing, chunking, caching |
| Python execution security | High | High | Sandboxing, expression validation, timeout |
| Charting library limitations | Medium | Medium | Evaluate alternatives, custom rendering |
| Third-party API changes | Low | Medium | Abstract integrations, version pinning |

### 8.2 Business Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Delayed launch | Medium | Medium | Agile methodology, MVP focus |
| Low initial adoption | Medium | High | Beta program, marketing, feedback loops |
| Competitor features | Medium | Medium | Focus on core differentiators |
| Budget overrun | Medium | Medium | Phased approach, regular cost reviews |

---

## 9. Success Metrics

### 9.1 Technical Metrics

| Metric | Target |
|--------|--------|
| API Response Time (P95) | < 200ms |
| Uptime | > 99.9% |
| Error Rate | < 0.1% |
| Deployment Frequency | Weekly |
| Lead Time for Changes | < 1 day |

### 9.2 Business Metrics (Post-Launch)

| Metric | Month 1 | Month 3 | Month 6 |
|--------|---------|---------|---------|
| Registered Users | 100 | 500 | 2,000 |
| Paying Customers | 10 | 50 | 200 |
| Monthly Revenue | €200 | €1,000 | €4,000 |
| Churn Rate | N/A | < 5% | < 3% |

---

## 10. Appendix

### 10.1 Technology Versions

| Technology | Version |
|------------|---------|
| .NET | 8.0 LTS |
| ASP.NET Core | 8.0 |
| Entity Framework Core | 8.0 |
| React | 18.x |
| TypeScript | 5.x |
| Node.js | 20.x LTS |
| Terraform | 1.6+ |
| Docker | 24.x |

### 10.2 Azure Services

| Service | SKU |
|---------|-----|
| Azure SQL Database | General Purpose (2-4 vCores) |
| Azure Container Apps | Consumption |
| Azure Functions | Consumption |
| Azure Blob Storage | GPv2 LRS |
| Azure Redis Cache | Standard C0-C1 |
| Azure AD B2C | Standard |
| Azure CDN | Standard Microsoft |

### 10.3 Key Contacts

| Role | Responsibility |
|------|----------------|
| Project Sponsor | Budget, business decisions |
| Tech Lead | Architecture, technical decisions |
| Product Manager | Features, priorities, stakeholders |
| DevOps Lead | Infrastructure, deployments |

---

## 11. Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2024-XX-XX | Solution Architect | Initial version |

---

*This implementation plan is a living document and should be updated as the project progresses.*
