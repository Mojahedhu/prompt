# one

/database-design , /database-architect
"Design a Production-Ready supabase database schema for a Multi-Tenant B2B School Management SaaS. The system supports four primary roles: Super Admin, School Admin, Teacher, and Student. Ensure every tenant-specific model includes an organizationId for data isolation. Define the relations between Organizations, Users, Classes, Subjects, Enrollments, and Attendance. Include necessary indexes for multi-tenant query performance."
as well as roles we have to create separate tables for teachers, students, super admins. and admin schooled. And every table must have its own RLS policy. So later, we can display all the students, teachers, and we must make relations between students and classes, teacher and classes. school admin and his related school. But the super admin has full access to do everything, so update the RLS policies according to this.

# 2- Stitch design

Design a scalable, responsive Web Application UI architecture for a B2B Multi-Tenant School Management Platform. The UI must support four main user perspectives: Platform Super Admin, School Administrator, Teacher, and Student. Focus strictly on layout logic, user workflows, and structural hierarchy.

Key Requirements & Screen Specifications:

1. Authentication & Tenant Onboarding Flow:

- Organization selection/creation switcher.
- Clean B2B subscription checkout interface showcasing tiered plans and feature gates.

2. School Administrator Dashboard (Focus: Operational Management):

- High-level metric cards (Total Enrolled Students, Active Teachers, Monthly Revenue/Subscription Status, Attendance Rate).
- Multi-tenant navigation sidebar dynamically displaying school-specific branding.
- Comprehensive data management layout (Data Grid) for Users & Staff featuring complex filtering, batch actions, role assignment, and pagination.

3. Teacher Workspace (Focus: Productivity & Execution):

- Interactive timetable and daily schedule grid.
- Rapid-action Attendance Marking interface optimized for minimal clicks.
- Gradebook and assignment submission tracking interface with status badges.

4. Student Portal (Focus: Simplicity & Engagement):

- Daily agenda view displaying upcoming classes, assignments, and announcements.
- Attendance history visualization and grade summary cards.

System Design Constraints:

- Layout Strategy: Standard SaaS Application Layout (Persistent Collapsible Sidebar, Top Global Header with Organization Switcher & User Profile, Main Content Area).
- Component Guidelines: High-density data tables, modal dialogs for quick updates, empty-state placeholders for unconfigured modules, and visible RBAC UI locks for features disabled under lower-tier subscription plans.
- UX Logic: Prioritize low-latency interactions, quick-action context menus, and inline data editing where applicable.

```

```

# 3- app, clerk, and strip integration

"Act as a Senior Software Engineer. Design a clean and high-level architectural flow for a B2B subscription system integrating Stripe and Clerk Organizations in a Next.js application.

the features inside free plan:
50_students
5_teachers
basic_attendance
1_branch

the features inside pro plan:
unlimited_students
unlimited_teachers
multiple_branches
ai_schedule_optimization
dedicated_audit_logs

Outline the Stripe Checkout creation flow that attaches the active Clerk organizationId to the checkout session.

Detail the Stripe Webhook logic to capture successful payments and failed payments and update the Clerk Organization's publicMetadata with the active plan name and status. and we must update and create all the payment status inside supabase"

```

```

# 4. Integration, clerk (user, organization) => app => supabase, stripe => app => supabase

Okay, guide me how we can save every user inside our Superbase hosted users as well as how to set up Stripe, like Webhook secret, publishable key, secret key, and everything about the integration between Stripe and Declerc. So whenever any organization upgrade a plan, we must see everything inside Superbase also. And we have to manage the webhooks.

```

```

# 5. test and test

Okay, guide me how we can save every user inside our Superbase hosted users as well as how to set up Stripe, like Webhook secret, publishable key, secret key, and everything about the integration between Stripe and Declerc. So whenever any organization upgrade a plan, we must see everything inside Superbase also. And we have to manage the webhooks.

## link

Modify the landing page. Let's introduce the users about our website and a button to create our first organization. In this space, you can create some mock data. Hirossection. and FAQ section and any other sections related to out website
