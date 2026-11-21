# Quantumberg Project Management Dashboard

A Frappe Desk dashboard for project managers and project teams using ERPNext. It combines project, task, timesheet, deadline, team workload, and project-finance information in one Vue 3 interface. Dashboard data is read live from the site's ERPNext records; this app does not define its own business DocTypes or store a separate copy of that data.

The app is developed for the `version-16` Frappe/ERPNext branch.

## Features

- Project and task status summaries, KPI cards, priority distribution, and activity trends.
- Searchable project and task lists, with links to the corresponding ERPNext records.
- Project deadline and milestone tracking, including overdue and upcoming work.
- Timesheet hours summarized by project for a selected date range.
- Project health, team workload, budget-versus-actual charts, cost breakdown, and accounting entries.
- Date-range selection; when no dates are supplied, the backend defaults to the last 365 days.
- Role-aware data and finance visibility:
  - System Managers and Administrators have administrator access.
  - Projects Managers have manager access.
  - Other dashboard users are scoped to projects where they are listed as a Project User or are the project owner.
  - Finance, accounting, and team details are limited to administrators and Projects Managers.
- Adapts to sites where optional DocTypes are unavailable by returning empty data for those sections.

The dashboard is available in Desk at **Project Management Dashboard** (route: `/app/project-management-dashboard`).

## Requirements

### App dependencies

Install these apps on the same bench, using compatible version 16 branches:

| Dependency app   | Required version | Purpose                                                                                    |
| ---------------- | ---------------- | ------------------------------------------------------------------------------------------ |
| Frappe Framework | Version 16       | Provides the Frappe runtime, Desk, and bench environment.                                  |
| ERPNext          | Version 16       | Required for the Project, Task, Timesheet, and accounting data displayed by the dashboard. |

Install ERPNext on the site before installing this dashboard. ERPNext is not currently declared in this app's `required_apps` hook, so Bench will not install it automatically.

### Development tools

- Python **3.14 or newer**, as specified in `pyproject.toml`.
- Node.js **24** and npm to install or build the Vue frontend (Node 24 is also used in CI).
- A working Frappe bench and site. Follow the [Frappe Bench installation guide](https://github.com/frappe/bench) to prepare one.

The frontend dependencies are declared in `frontend/package.json`: Vue, ApexCharts, Axios, Lucide Vue icons, Tailwind CSS, Vite, and their Vue/Tailwind Vite plugins. `npm install` in `frontend/` installs these packages; no separate global frontend packages are required.

## Installation

From an existing bench directory, fetch the app from this repository's `version-16` branch:

```bash
cd /path/to/frappe-bench
bench get-app https://github.com/erp-quantumberg/erpnext-project_management-quantumberg_project_management_dashboard.git --branch version-16
```

Ensure ERPNext version 16 is installed on the bench and site. If it is not already installed, fetch the matching ERPNext branch and install it before this dashboard:

```bash
bench get-app https://github.com/frappe/erpnext.git --branch version-16
bench --site your-site.example install-app erpnext
```

Install the dashboard and build its frontend assets:

```bash
bench --site your-site.example install-app quantumberg_project_management_dashboard
cd apps/quantumberg_project_management_dashboard/frontend
npm install
npm run build
cd ../../..
bench build --app quantumberg_project_management_dashboard
bench --site your-site.example migrate
```

Replace `your-site.example` with the site name shown under the bench's `sites/` directory. Sign in to Desk with a user who has a dashboard role, then open **Project Management Dashboard** from the Desk search or visit `/app/project-management-dashboard`.

## Setup and frontend development

The dashboard frontend is in `frontend/`. Its Vite configuration expects this repository to be inside a Frappe bench app directory and reads `sites/common_site_config.json` to configure the development proxy. Start the bench from one terminal:

```bash
cd /path/to/frappe-bench
bench start
```

Then start the Vite development server from another terminal:

```bash
cd /path/to/frappe-bench/apps/quantumberg_project_management_dashboard/frontend
npm install
npm run dev
```

The development server listens on port `8080` and proxies API, asset, and file requests to the Frappe web server. For a production build, run `npm run build` in `frontend/`; it writes the bundled files to `quantumberg_project_management_dashboard/public/dashboard/`. Rebuild those assets after frontend changes, then run `bench build --app quantumberg_project_management_dashboard`.

## Configuration notes

- There are no app-specific site settings to configure. The dashboard reads existing ERPNext records and uses the site's global default currency (falling back to INR if it is not set).
- The default date range is the last 365 days. The dashboard sends `from_date` and `to_date` to the `get_dashboard_data` API method when a range is selected.
- The Desk page is configured for the `Projects User`, `Projects Manager`, and `System Manager` roles. Backend access also recognizes `Administrator`.
- Project users see only projects where they are a member in the Project Users table or are the project owner. Projects Managers and administrators can see all projects.
- Budget, actual-cost, billing, accounting-entry, and team-workload information is restricted to Projects Managers and administrators. Assign suitable ERPNext roles and project memberships to users; do not rely on hiding dashboard sections as a substitute for server-side permissions.
- Accounting entries are read from ERPNext's General Ledger Entry data. The app does not create accounting transactions.
- The frontend communicates with the whitelisted Frappe method `quantumberg_project_management_dashboard.api.get_dashboard_data`.

## Troubleshooting

| Problem                                                               | Checks and remedies                                                                                                                                                                                     |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The dashboard page is missing or access is denied                     | Confirm the app is installed on the site, the user has one of the configured Desk page roles, and the user can sign in to Desk. Run `bench --site your-site.example list-apps` to check installed apps. |
| Dashboard sections are empty                                          | Confirm ERPNext is installed and the site contains Project, Task, or Timesheet records relevant to the selected date range. A project user must be a Project User member or project owner.              |
| Financial or team sections are hidden                                 | These sections are restricted to Projects Managers, System Managers, and Administrators. Check the user's roles and reload the page.                                                                    |
| Vite fails to start because `common_site_config.json` cannot be found | Run the frontend from the `frontend/` directory inside the installed bench app, and verify that the bench `sites/common_site_config.json` exists.                                                       |
| API requests fail in development                                      | Start `bench start`, check the configured Frappe web-server port in `sites/common_site_config.json`, and verify that the logged-in browser session can access the site.                                 |
| Frontend changes do not appear in Desk                                | Run `npm run build` from `frontend/`, then `bench build --app quantumberg_project_management_dashboard`. Hard-refresh the browser to bypass cached assets.                                              |
| Build reports missing frontend packages                               | Run `npm install` from `apps/quantumberg_project_management_dashboard/frontend/` and retry the build.                                                                                                   |

## Contributing

Contributions are welcome. Keep changes focused, follow the existing Python and Vue conventions, and include tests for behavior changes where appropriate.

1. Create a branch for your change.
2. Install frontend packages with `npm install` in `frontend/` if your change requires building or checking the frontend.
3. Install and enable pre-commit from the app directory:

   ```bash
   pip install pre-commit
   cd /path/to/frappe-bench/apps/quantumberg_project_management_dashboard
   pre-commit install
   ```

4. Build the frontend after making frontend changes:

   ```bash
   cd frontend
   npm run build
   ```

5. Run the app tests on a configured Frappe site:

   ```bash
   cd /path/to/frappe-bench
   bench --site your-site.example run-tests --app quantumberg_project_management_dashboard
   ```

The repository's pre-commit configuration includes Ruff, ESLint, Prettier, and basic file checks. GitHub Actions also defines CI and linter workflows.

## License

This project declares the MIT License. See [license.txt](license.txt) for the license text.
