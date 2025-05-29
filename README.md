(In english below)

GitOps projekt áttekintése
Ez a README fájl tömör bevezetést nyújt a GitOps-ba, és felvázolja a CI/CD folyamat GitHub Actions, AWS, Terraform, Docker és Helm használatával történő beállításának architektúráját és lépéseit.

Mi a GitOps?
A GitOps egy DevOps-on belüli stratégia, amely a hangsúlyt a Git (egy verziókövető rendszer) által kezelt kódként történő tárolásra helyezi. A változtatások alkalmazása kizárólag a Giten keresztül történik, biztosítva a nyomon követhetőséget és a konzisztenciát. Megoldja a változtatások nyomon követésének gyakori automatizálási problémáját, és megakadályozza a manuális módosítások okozta infrastruktúra-eltolódást. Egy mikroszolgáltatás-architektúrában a GitOps segít kezelni a gyakori alkalmazás- és infrastruktúra-változások összetettségét.

A lényeg az, hogy ahelyett, hogy a mérnökök közvetlenül módosítanák az infrastruktúrát, minden változtatást kódként rögzítenek a Gitben. Ezeket a változtatásokat ezután a GitOps eszközök (mint például a GitHub Actions, ArgoCD, Tekton, Jenkins X) automatikusan alkalmazzák, biztosítva a nyomon követhetőséget, a konzisztenciát és kiküszöbölve a manuális hibákat.

Projekt áttekintése
Ez a projekt egy CI/CD folyamatot mutat be, amely két GitHub adattárral és GitHub Actions munkafolyamatokkal épült, megkönnyítve az infrastruktúra és egy alkalmazás automatizált telepítését az AWS-be. A fejlesztés elsősorban VS Code használatával történik.

Architektúra
A projekt két fő GitHub adattárat használ:

infrastructure-repo: Terraform kódot tartalmaz az infrastruktúra kiépítéséhez.
application-repo: Az alkalmazás kódját tárolja (Java, Docker, Helm).
Minden adattárnak saját GitHub Actions munkafolyamata van.

Az AWS-ben kiépített infrastruktúra-összetevők a következők:

Amazon EKS (Kubernetes klaszter)

Amazon ECR (Docker képadattár)

VPC alhálózat (hálózat)

Munkafolyamatok összefoglalása
1. Infrastruktúra munkafolyamat (Terraform)
Ezt a munkafolyamatot az infrastructure-repo kezeli a GitHub Actions segítségével.

Lépések:

Fejlesztés: A kódmódosítások a VS Code-ban kerülnek végrehajtásra, és egy átmeneti ágba kerülnek. GitHub Actions munkafolyamat (Staging):
A terraform validate futtatása.
A terraform plan végrehajtása a változtatások felülvizsgálatára azok alkalmazása nélkül.
Pull Request: A pull request létrehozása a stagingből a main ágba.
Approval & Merge: A pull request felülvizsgálata, jóváhagyása és a main ágba való egyesítése.
Deploy: A main ágba való egyesüléskor a terraform apply automatikusan végrehajtásra kerül, kiépítve az AWS VPC-t és az EKS klasztert.

2. Alkalmazás munkafolyamat (Build, Test & Deploy)
Ezt a munkafolyamatot az alkalmazástárházon belül kezelik a GitHub Actions, a SonarCloud, a Maven, a Docker és a Helm segítségével.

Lépések:

Fejlesztés: A kódmódosítások véglegesítése a VS Code-ból, és a GitHubba küldése.

GitHub Actions munkafolyamat:
Build & Test: A Maven teszteket épít és futtat.
Minőségelemzés: A SonarCloud minőségellenőrzésként kódelemzést végez.
Docker Build: Egy Docker képfájl létrehozása.
Pust Image: A Docker képfájl elküldése az Amazon ECR-be.
Telepítés: A rendszerképet egy Helm-diagramon keresztül telepítik az EKS-klaszterre, automatikus rendszerkép-címkefrissítésekkel.

Kulcsfontosságú gyakorlatok
Elágazási stratégia: Minden fejlesztés egy szakaszos ágban történik. Miután a változtatások tesztelésre kerültek és készen állnak, azokat a fő ágba egyesítik. A fő ág felelős a terraform alkalmazási műveletekért és az infrastruktúra-változásokért.

GitHub titkok: Az olyan érzékeny információkat, mint az AWS hozzáférési kulcsok, az S3 tárolónevek és az ECR regisztrációs URL-ek, biztonságosan tárolják GitHub titkokként mind az iac-vprofile, mind a vprofile-action tárolókban.

Terraform bevált gyakorlatok: A munkafolyamat magában foglalja a terraform fmt, validálás és tervezési lépéseket a kódminőség biztosítása és a változtatások megértése érdekében az alkalmazás előtt. A terraform alkalmazás csak a fő ágon fut, hogy megakadályozza a véletlen infrastruktúra-változásokat.

Automatizált telepítés: A teljes CI/CD folyamat, a kód véglegesítésétől az alkalmazás EKS-en történő telepítéséig, automatizált a GitHub Actions segítségével.

Dinamikus címkézés: A Docker rendszerképeket a legújabb és a github.run_number címkékkel látják el a verziókövetés érdekében. Helm-diagramok: A Kubernetes telepítését Helm-diagramok segítségével kezelik a paraméteres és konzisztens telepítések érdekében. A rendszerkép-címkék dinamikusan kerülnek átadásra a Helm-diagramoknak a telepítés során.

===========================================================================================================================================

GitOps Project Overview
This README provides a concise introduction to GitOps and outlines the architecture and steps involved in setting up a CI/CD pipeline using GitHub Actions, AWS, Terraform, Docker, and Helm.

What is GitOps?
GitOps is a strategy within DevOps that emphasizes storing everything as code, managed by Git (a version control system). Changes are applied exclusively through Git, ensuring traceability and consistency. It solves the common automation problem of tracking changes and prevents infrastructure drift caused by manual modifications. In a microservice architecture, GitOps helps manage the complexity of frequent application and infrastructure changes.

The core idea is that instead of engineers directly modifying infrastructure, all changes are recorded in Git as code. These changes are then automatically applied by GitOps tools (like GitHub Actions, ArgoCD, Tekton, Jenkins X), ensuring traceability, consistency, and eliminating manual errors.

Project Overview
This project demonstrates a CI/CD pipeline built with two GitHub repositories and GitHub Actions workflows, facilitating the automated deployment of infrastructure and an application to AWS. Development is primarily done using VS Code.

Architecture
The project utilizes two main GitHub repositories:

infrastructure-repo: Contains Terraform code for infrastructure provisioning.
application-repo: Houses the application code (Java, Docker, Helm).
Each repository has its own GitHub Actions workflow.

The infrastructure components provisioned in AWS include:

Amazon EKS (Kubernetes Cluster) 

Amazon ECR (Docker image repository) 
VPC subnet (network) 
Workflow Summaries
1. Infrastructure Workflow (Terraform)
This workflow is managed within the infrastructure-repo using GitHub Actions.

Steps:

Development: Code changes are made in VS Code and pushed to a staging branch.
GitHub Actions Workflow (Staging):
terraform validate is run.
terraform plan is executed to review changes without applying them.
Pull Request: A Pull Request is created from staging to main.
Approval & Merge: The Pull Request is reviewed, approved, and merged into the main branch.
Deploy: Upon merging to main, terraform apply is automatically executed, provisioning AWS VPC and the EKS Cluster.
2. Application Workflow (Build, Test & Deploy)
This workflow is managed within the application-repo using GitHub Actions, SonarCloud, Maven, Docker, and Helm.

Steps:

Development: Code changes are committed from VS Code and pushed to GitHub.
GitHub Actions Workflow:
Build & Test: Maven builds and runs tests.
Quality Analysis: SonarCloud performs code analysis as a quality gate.
Docker Build: A Docker image is built.
Push Image: The Docker image is pushed to Amazon ECR.
Deploy: The image is deployed to the EKS Cluster via a Helm Chart, with automatic image tag updates.
Key Practices
Branching Strategy: All development is done in a stage branch. Once changes are tested and ready, they are merged into the main branch. The main branch is responsible for terraform apply operations and infrastructure changes.

GitHub Secrets: Sensitive information like AWS access keys, S3 bucket names, and ECR registry URLs are stored securely as GitHub Secrets in both iac-vprofile and vprofile-action repositories.

Terraform Best Practices: The workflow includes terraform fmt, validate, and plan steps for ensuring code quality and understanding changes before applying. The terraform apply is only executed on the main branch to prevent accidental infrastructure changes.

Automated Deployment: The entire CI/CD process, from code commit to application deployment on EKS, is automated through GitHub Actions.
Dynamic Tagging: Docker images are tagged with latest and github.run_number for version control.
Helm Charts: Kubernetes deployment is managed using Helm charts for parameterized and consistent deployments. Image tags are dynamically passed to the Helm charts during deployment.
