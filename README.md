# TimeSeriesViewer

TimeSeriesViewer is a cloud-native SaaS application designed for analyzing and visualizing time series data. Built with a C# backend on Azure infrastructure and a React frontend, it provides enterprise-grade reliability, scalability, and security for engineers, analysts, and researchers.

## Key Features

- **File Support**: Open and visualize time-series files (CSV, Apache Parquet, and more)
- **Time Axis Normalization**: Normalize time-axis across multiple files with different sampling rates and start times
- **Multi-File Visualization**: Show columns from multiple files in the same charts
- **Flexible Chart Layouts**: Display columns from the same file in different charts
- **Data Transformations**: Manipulate column values using Python syntax for mathematical operations
- **Project Management**: Save and manage projects including column setups, files, and transformations
- **Export Capabilities**: Export charts in various formats (PNG, PDF, SVG)
- **Version Control**: Version projects and save different configurations
- **Authentication**: Support for popular identity providers (Google, Apple, Microsoft)
- **Payments**: Subscription management via Stripe (European market optimized)

## Documentation

Comprehensive documentation is available in the `/docs` directory:

1. **[Technology Choices and Solution Summary](docs/01-technology-choices-and-solution-summary.md)**  
   Overview of technology stack, high-level architecture, and design decisions.

2. **[Solution and Software Architecture](docs/02-solution-and-software-architecture.md)**  
   Detailed technical architecture including component design, data models, security, and deployment.

3. **[Implementation Plan](docs/03-implementation-plan.md)**  
   Phased implementation roadmap with sprints, tasks, and resource requirements.

## Technology Stack

### Backend
- .NET 8 (LTS)
- ASP.NET Core Web API
- Azure Functions (serverless processing)
- Entity Framework Core
- SignalR (real-time updates)

### Frontend
- React 18+ with TypeScript
- Redux Toolkit / Zustand (state management)
- Apache ECharts (charting)
- Material-UI (component library)
- Vite (build tool)

### Infrastructure
- Azure SQL Database
- Azure Blob Storage
- Azure Cache for Redis
- Azure AD B2C (authentication)
- Azure Container Apps / AKS
- Azure API Management

### DevOps
- GitHub Actions (CI/CD)
- Terraform / Bicep (IaC)
- Azure Application Insights (monitoring)

## Getting Started

*Development setup instructions will be added as the project progresses.*

## Project Structure

```
TimeSeriesViewer/
├── docs/                    # Architecture and planning documents
├── infrastructure/          # Terraform/Bicep infrastructure code
├── src/
│   ├── backend/            # .NET backend services
│   └── frontend/           # React frontend application
├── docker/                 # Docker configuration
└── README.md
```

## License

*License information to be added.*

## Contributing

*Contributing guidelines to be added.*
