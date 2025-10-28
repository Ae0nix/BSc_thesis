# Makefile per eseguire kubectl port-forward sul pod di Grafana.
# Trova automaticamente il nome del pod nel namespace specificato usando la label corretta.

# --- Variabili Configurabili ---
NAMESPACE   := my-grafana
# La label usata per trovare il pod (in questo caso, 'app=grafana')
APP_LABEL_KEY := app
APP_LABEL_VAL := grafana
ADDRESS     := 0.0.0.0
LOCAL_PORT  := 3000
REMOTE_PORT := 3000
KUBECTL     := kubectl

# --- Logica Interna (non modificare) ---
# Trova il nome del primo pod che corrisponde alla label specificata.
POD_NAME := $(shell $(KUBECTL) get pods --namespace=$(NAMESPACE) --selector=$(APP_LABEL_KEY)=$(APP_LABEL_VAL) -o name --no-headers | head -n 1 | sed 's/pod\///')

# --- Target ---
.PHONY: port-forward help

port-forward:
	@if [ -z "$(POD_NAME)" ]; then \
		echo "Errore: Nessun pod trovato con la label '$(APP_LABEL_KEY)=$(APP_LABEL_VAL)' nel namespace '$(NAMESPACE)'."; \
		exit 1; \
	fi
	@echo "Trovato pod Grafana: $(POD_NAME)"
	@echo "Avvio port-forward su $(ADDRESS):$(LOCAL_PORT) -> pod $(REMOTE_PORT)..."
	$(KUBECTL) port-forward pod/$(POD_NAME) --namespace=$(NAMESPACE) --address $(ADDRESS) $(LOCAL_PORT):$(REMOTE_PORT)

help:
	@echo "Uso:"
	@echo "  make port-forward    - Avvia il port-forward per Grafana."
	@echo ""
	@echo "Variabili che puoi sovrascrivere:"
	@echo "  NAMESPACE     (default: $(NAMESPACE))"
	@echo "  APP_LABEL_VAL (default: $(APP_LABEL_VAL))"
	@echo "  LOCAL_PORT    (default: $(LOCAL_PORT))"

