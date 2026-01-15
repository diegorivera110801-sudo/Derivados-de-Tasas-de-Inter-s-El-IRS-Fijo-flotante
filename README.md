# Derivados-de-Tasas-de-Inter-s-El-IRS-Fijo-flotante
En este trabajo hago un ejemplo de como se veria reflejado un derivado de la tasa de interés de la FED en diversos escenarios dependiendo de las expeculaciones del mercado y de esta formadar seguridad a los acreedores del Swap.
# ==========
# IRS-pago flotante/recibo fijo( por bajadas de la FED)
# =========

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
# ====== Inputs del swap ======
N = 100_000_000      # Notional (base de cálculo; NO se intercambia)
delta = 1.0          # Δ: año-fracción entre pagos (1 = anual)
T = 5                # Tenor del swap (años): pagos en t=1..T

# ====== Curvas (t0 y t1) ======
# z_t0: curva en la fecha del contrato (sirve para fijar K at-par)
# z_t1: curva en la fecha de valuación (sirve para revaluar el swap)
z_t0 = np.array([0.0422, 0.0413, 0.0409, 0.0411, 0.0407])  # ejemplo 2024
z_t1 = np.array([0.0363, 0.0361, 0.0362, 0.0365, 0.0378])  # ejemplo 2025 (bajada)

t = np.arange(1, T+1)  # [1,2,3,4,5]
DF_t0 = np.exp(-z_t0 * t)
DF_t1 = np.exp(-z_t1 * t)

df_curve = pd.DataFrame({
    "t": t,
    "z_t0": z_t0,
    "z_t1": z_t1,
    "DF_t0": DF_t0,
    "DF_t1": DF_t1
})

df_curve
annuity_t0 = np.sum(DF_t0 * delta)
K = (1 - DF_t0[-1]) / annuity_t0

annuity_t0, K
DF_ext = np.r_[1.0, DF_t1]                # [DF(0)=1, DF(1),...,DF(T)]
forward = (DF_ext[:-1] / DF_ext[1:] - 1) / delta

df_curve["Forward_t1"] = forward
df_curve
CF_fixed = N * K * delta
CF_float = N * forward * delta

cashflows = pd.DataFrame({
    "t": t,
    "DF_t1": DF_t1,
    "Forward_t1": forward,
    "CF_fixed": CF_fixed,
    "CF_float": CF_float
})

cashflows
cashflows["PV_fixed"] = cashflows["CF_fixed"] * cashflows["DF_t1"]
cashflows["PV_float"] = cashflows["CF_float"] * cashflows["DF_t1"]

PV_fixed_total = cashflows["PV_fixed"].sum()
PV_float_total = cashflows["PV_float"].sum()

PV_fixed_total, PV_float_total
NPV_receive_fixed_pay_float = PV_fixed_total - PV_float_total
NPV_receive_fixed_pay_float
NPV_pay_fixed_receive_float = PV_float_total - PV_fixed_total
NPV_pay_fixed_receive_float
#si no te cubres pierdes esto, el IRS hace que te cubras de futuras bajas o subidad de la FED.
plt.figure()
plt.plot(t, z_t0, marker="o", label="Curva t0")
plt.plot(t, z_t1, marker="o", label="Curva t1")
plt.title("Curvas (z): t0 vs t1")
plt.xlabel("Tenor (años)")
plt.ylabel("Tasa")
plt.legend()
plt.show()

plt.figure()
plt.plot(t, forward, marker="o")
plt.title("Forwards implícitos (desde DF en t1)")
plt.xlabel("Periodo (año)")
plt.ylabel("Forward")
plt.show()

plt.figure()
plt.bar(t, cashflows["PV_fixed"] - cashflows["PV_float"])
plt.title("Contribución por año al NPV (Receive Fixed / Pay Float)")
plt.xlabel("Año")
plt.ylabel("PV(fijo) - PV(flotante)")
plt.show()
