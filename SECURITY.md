Branching Strategies GuideA branching strategy defines how a software development team uses Git branches to organize, develop, test, and release code. Choosing the right strategy depends on your team's size, release frequency, compliance requirements, and architecture (e.g., single web service vs. multi-tenant monorepo).Overview & ComparisonStrategyBest Suited ForComplexityRelease CadenceWeb Services (Trunk-Based)Continuous delivery, single SaaS web servicesLowHigh / ContinuousLong-Lived Release BranchesVersioned software (on-prem, mobile, multi-version support)MediumScheduled / PeriodicBranch per EnvironmentStage-gated pipelines, multi-team dependencies, V-ModelHighRelease Candidate (RC) gates1. Web Services Strategy (Trunk-Based / Simple Feature Branches)This strategy follows standard continuous delivery principles. The main branch represents the live production state. All feature development occurs on short-lived feature branches that merge directly back into main.Workflow DiagramgitGraph
    commit id: "Initial (v1.0)" tag: "v1.0"
    branch feature-1
    checkout feature-1
    commit id: "start feature-1"
    branch feature-2
    checkout feature-2
    commit id: "start feature-2"
    checkout feature-1
    commit id: "refine feature-1"
    checkout main
    merge feature-1 id: "merge feature-1" tag: "v1.0.1"
    checkout feature-2
    commit id: "build feature-2"
    merge main id: "sync main"
    checkout main
    merge feature-2 id: "merge feature-2" tag: "v1.1"
Key Workflow RulesShort-Lived Feature Branches: Create branches (feature-1, feature-2) directly from main.Frequent Synchronization: Rebase or merge updates from main into active feature branches to prevent drift.Direct Merge & Release: Once code reviews and CI/CD checks pass, feature branches merge into main and automated deployment cuts a release tag (e.g., v1.1).2. Long-Lived Release Branches StrategyUsed when supporting multiple production versions simultaneously or when releases require an extended release candidate (RC) stabilization period.Workflow DiagramgitGraph
    commit id: "v1.0 Base" tag: "v1.0"
    branch release-2.0
    checkout release-2.0
    commit id: "2.0 RC 1"
    checkout main
    commit id: "Hotfix: Security Bug"
    checkout release-2.0
    commit id: "2.0 RC 2"
    checkout main
    commit id: "Hotfix: Performance Bug"
    checkout release-2.0
    commit id: "2.0 RC 3"
    checkout main
    merge release-2.0 id: "Merge 2.0 to Main" tag: "v2.0"
Key Workflow RulesRelease Branch Isolation: Major feature sets are stabilized on dedicated long-lived branches (e.g., release-2.0).Hotfix Flow: Critical production fixes land on main and are cherry-picked or merged back into active release candidate branches.RC Tagging: Iterative Release Candidates (2.0 RC 1, 2.0 RC 2) undergo testing until stability criteria are met.3. Branch per Environment StrategyCommonly applied in enterprise organizations requiring strict compliance, manual QA/UAT verification phases, or waterfall/V-model approval structures.Workflow DiagramgitGraph
    commit id: "v1.0 Base" tag: "v1.0"
    branch test
    branch UAT
    checkout main
    branch feature-1
    checkout feature-1
    commit id: "Start Feature"
    commit id: "Develop Feature"
    checkout main
    merge feature-1 id: "Merge Feature 1"
    checkout test
    merge main id: "Promote to Test (v1.1 RC1)"
    checkout UAT
    merge test id: "Promote to UAT"
    checkout main
    commit id: "Promote to Production" tag: "v1.1"
Key Workflow RulesSequential Promotion: Code moves sequentially through environment-bound branches (feature $\rightarrow$ main $\rightarrow$ test $\rightarrow$ UAT $\rightarrow$ production).Stage Gating: Pushing or merging to an environment branch automatically triggers deployment pipelines for that specific target environment.Governance & Security Best PracticesProtected Branches: Restrict direct commits to main, release/*, and environment branches.Automated Approvals: Enforce minimum code reviewer approvals and require passing status checks (unit/integration tests, SAST scanning) before merging.Merged Result Pipelines: Run CI/CD pipelines on the predicted merged state to ensure zero regression prior to landing code on main.
