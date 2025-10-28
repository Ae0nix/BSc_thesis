# Makefile for setting up and managing the InfluxDB instance for tests.

# --- Configuration ---
# You can change these variables if needed.
NAMESPACE    := influx
RELEASE_NAME := local-influx
CHART        := influxdata/influxdb2
PVC_NAME     := $(RELEASE_NAME)-influxdb2
PV_NAME      := pv-influx
LOCAL_PORT   := 8086
REMOTE_PORT  := 80

# --- Targets ---

# Using .PHONY tells make that these targets don't create files.
.PHONY: all deploy-influx uninstall clean credentials port-forward setup

# The default command when you run `make`.
all: deploy-influx

# Installs InfluxDB and waits for it to be ready.
deploy-influx:
	@echo "--- Creating persistent volume ---"
	@kubectl apply -f /root/pods/storage/pv-volume.yaml
	@echo "--- 📦 Adding InfluxDB Helm repository and updating... ---"
	@helm repo add influxdata https://helm.influxdata.com/ > /dev/null
	@helm repo update > /dev/null
	@echo "--- 📂 Creating namespace: $(NAMESPACE)... ---"
	@kubectl create ns $(NAMESPACE) --dry-run=client -o yaml | kubectl apply -f - > /dev/null
	@echo "--- 🚀 Installing InfluxDB release '$(RELEASE_NAME)'... ---"
	@helm upgrade --install -n $(NAMESPACE) $(RELEASE_NAME) \
		--set persistence.enabled=true,persistence.size=9Gi,persistence.storageClass=manual,nodeSelector."kubernetes\.io/hostname"=w01 \
		$(CHART)
	@echo "--- ⏳ Waiting for InfluxDB pod to be ready... (this might take a minute) ---"
	@kubectl wait --for=condition=ready pod \
		-l app.kubernetes.io/name=influxdb2 \
		-n $(NAMESPACE) --timeout=300s
	@echo "--- ✅ InfluxDB is installed and ready. ---"
	@make credentials

# Shows the admin credentials for InfluxDB.
credentials:
	@echo "\n--- 🔑 Your Admin Credentials ---"
	@echo "Password:"
	@kubectl get secret $(RELEASE_NAME)-influxdb2-auth -o "jsonpath={.data['admin-password']}" -n $(NAMESPACE) | base64 --decode
	@echo "\nToken:"
	@kubectl get secret $(RELEASE_NAME)-influxdb2-auth -o "jsonpath={.data['admin-token']}" -n $(NAMESPACE) | base64 --decode
	@echo "\n------------------------------------"

# Forwards the service port to your local machine.
# This command is blocking, run it in a separate terminal.
port-forward:
	@echo "--- 🌐 Starting port-forward to InfluxDB UI on http://localhost:$(LOCAL_PORT) ---"
	@echo "---    (Press Ctrl+C to stop)    ---"
	@kubectl port-forward --address 0.0.0.0 -n $(NAMESPACE) svc/$(RELEASE_NAME)-influxdb2 $(LOCAL_PORT):$(REMOTE_PORT)

# Runs the setup script to create the required organization and buckets.
# Assumes port-forwarding is active in another terminal.
setup:
	@echo "--- ⚙️ Setting up InfluxDB organization and buckets... ---"
	@export INFLUX_URL=http://localhost:$(LOCAL_PORT); \
	@export INFLUX_TOKEN=$$(kubectl get secret local-influx-influxdb2-auth -n influx -o "jsonpath={.data['admin-token']}" | base64 --decode); \
	if [ -z "$$INFLUX_TOKEN" ]; then \
		echo "Error: Failed to retrieve INFLUX_TOKEN. Is InfluxDB installed?"; \
		exit 1; \
	fi; \
	/root/serving-knative-v1.13.2/test/performance/visualization/setup-influx-db.sh
	@echo "--- OUTPUT SCRIPT SETUP_INFLUX_DB: $? "
	@echo "--- ✅ Setup script completed. ---"

# Completely removes the InfluxDB installation.
uninstall:
	@echo "--- 💣 Uninstalling Helm release '$(RELEASE_NAME)'... ---"
	@helm uninstall $(RELEASE_NAME) -n $(NAMESPACE)
	@echo "--- 💾 Deleting Persistent Volume Claim '$(PVC_NAME)'... ---"
	@kubectl delete pvc $(PVC_NAME) -n $(NAMESPACE) --ignore-not-found=true
	@echo "--- 💾 Deleting Persistent Volume '$(PVC_NAME)'... ---"
	@kubectl delete pv $(PV_NAME) -n $(NAMESPACE) --ignore-not-found=true
	@echo "--- 🗑️ Deleting namespace '$(NAMESPACE)'... ---"
	@kubectl delete ns $(NAMESPACE) --ignore-not-found=true
	@echo "--- ✅ Cleanup complete. ---"

# 'clean' is a common alias for 'uninstall'.
clean: uninstall
