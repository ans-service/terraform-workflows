# Plantillas de Workflows para Terraform

Workflows reutilizables de GitHub Actions para aprovisionar y destruir infraestructura con Terraform sobre Azure. Al referenciar estas plantillas desde otros repositorios mantenemos la automatización alineada y centralizamos configuraciones de tokens, workspaces y registros.

## Workflows

### Provisioning (`.github/workflows/provisioning.yaml`)
- Formatea el código Terraform, configura credenciales Git para módulos privados e inicializa el backend.
- Verifica/crea el workspace solicitado, ejecuta `terraform validate` y genera un plan en los pull requests (publica el resumen en la PR).
- Aplica automáticamente cuando hay push a `main`, usando `${WORKSPACE_NAME}.tfvars` más los flags adicionales ingresados por `ARGS`.

### Destroy (`.github/workflows/destroy.yaml`)
- Reutiliza la misma autenticación, asegura el workspace y ejecuta `terraform destroy -auto-approve`.

## Secretos requeridos
- `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`: credenciales del service principal de Azure.
- `GIT_TOKEN`: token con acceso al repositorio/organización donde viven los módulos Terraform.
- `ORGANIZATION_NAME`: nombre de la organización GitHub usado para reescribir URLs de módulos.

## Inputs
- `WORKSPACE_NAME` (string, obligatorio): nombre del workspace/entorno Terraform. Debe contar con un archivo `*.tfvars` equivalente durante el aprovisionamiento.
- `ARGS` (string, opcional, solo provisioning): flags adicionales que se agregan a `terraform plan`/`apply` (ej. `-target=module.example`).

## Ejemplo de uso

Crea un workflow en tu repo de Terraform que invoque la plantilla mediante `workflow_call`:

```yaml
name: Desplegar Infraestructura

on:
	push:
		branches: [ main ]
	pull_request:

jobs:
	terraform:
		uses: ans-service/terraform-workflows/.github/workflows/provisioning.yaml@main
		with:
			WORKSPACE_NAME: prod
			ARGS: "-parallelism=5"
		secrets:
			AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
			AZURE_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
			AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
			AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
			GIT_TOKEN: ${{ secrets.MODULE_TOKEN }}
			ORGANIZATION_NAME: ${{ secrets.GH_ORG }}
```

Agrega un job similar apuntando a `destroy.yaml` cuando necesites automatizar la destrucción.

## Recomendaciones
- Mantén los nombres de workspace alineados con los archivos tfvars (`prod.tfvars`, `qa.tfvars`, etc.).
- Usa `ARGS` solo con flags soportados por plan y apply; si necesitas configuraciones distintas, crea workflows separados.
- Fija la versión de la plantilla (`@<git-sha>`) para asegurar reproducibilidad en repos dependientes.