# 🔵🟢 Blue-Green Deployment Strategy on Kubernetes (KIND)

Blue-Green deployment ek strategy hai jo Kubernetes (aur dusre environments) mein **downtime aur risk kam karne** ke liye use hoti hai jab application ka naya version deploy kiya jata hai.

Is mein do alag environments maintain kiye jate hain: **Blue** aur **Green**.

> **Author:** Waqas Saleem — DevOps Engineer
> **GitHub:** [Ranawaqas323421](https://github.com/Ranawaqas323421)

---

## 📌 Table of Contents

- [How It Works](#-how-it-works)
- [Pros & Cons](#-pros--cons)
- [Prerequisites](#-prerequisites)
- [Implementation Steps](#-implementation-steps)
- [Troubleshooting](#-troubleshooting)
- [Cleanup](#-cleanup)
- [Author](#-author)

---

## ⚙️ How It Works

| Stage | Description |
|-------|-------------|
| **Blue Environment** | Live environment jahan application ka current version chal raha hota hai. |
| **Green Environment** | Naya version yahan deploy hota hai. Is dauran Blue environment use karne walay users par koi asar nahi padta. |
| **Test Green** | Green environment up hone ke baad us ko fully test kiya jata hai. |
| **Switch Traffic** | Green stable validate hone ke baad traffic Blue se Green par switch kar diya jata hai. Ab Green hi production hai. |

---

## ✅ Pros & Cons

| 👍 Pros | 👎 Cons |
|---------|---------|
| Instant rollout / rollback | Double resources ki zaroorat hoti hai |
| Versioning issues se bachao — poora cluster state ek hi baar mein change hota hai | Production release se pehle poore platform ka proper test zaroori hai |

> [!NOTE]
> Ye deployment strategy **Production environment** ke liye suitable hai.

---

## 🧰 Prerequisites

- EC2 Instance (Ubuntu OS)
- Docker installed & configured
- KIND installed
- kubectl installed
- KIND cluster running (repo ki root directory mein maujood `kind-config.yml` use karein)

> [!NOTE]
> Repo ki **root directory** ke andar ja kar ye command chalayen:

```bash
kind create cluster --config kind-config.yml --name dep-strg
```

---

## 🚀 Implementation Steps

### Step 1: Namespace banayein

```bash
kubectl apply -f blue-green-ns.yml
```

### Step 2: Dono deployment manifests apply karein

```bash
kubectl apply -f online-shop-without-footer-blue-deployment.yaml
kubectl apply -f online-shop-green-deployment.yaml
```

### Step 3: Pods monitor karein

Naya terminal tab kholein aur watch command chalayen:

```bash
watch kubectl get pods -n blue-green-ns
```

Is se **Blue** (online shop *without footer*) aur **Green** (online shop *with footer* — naya feature) dono deploy ho jayenge.

### Step 4: Namespace ke saare resources check karein

```bash
kubectl get all -n blue-green-ns
```

### Step 5: Blue service ko port-forward karein

```bash
kubectl port-forward --address 0.0.0.0 svc/online-shop-blue-deployment-service 30001:3001 -n blue-green-ns &
```

EC2 Instance mein **port 30001** ka inbound rule open karein, phir browser mein check karein:

```
http://<Your_Instance_Public_Ip>:30001
```

### Step 6: Green service ko port-forward karein

```bash
kubectl port-forward --address 0.0.0.0 svc/online-shop-green-deployment-service 30000:3000 -n blue-green-ns &
```

EC2 Instance mein **port 30000** ka inbound rule open karein, phir check karein:

```
http://<Your_Instance_Public_Ip>:30000
```

> [!NOTE]
> URL aur port number dhyan se check karein.

### Step 7: Traffic Blue se Green par switch karein

`online-shop-without-footer-blue-deployment.yaml` file kholein aur **Service ke `selector`** field ko `online-shop-green` selector se replace karein.

Phir manifest dobara apply karein:

```bash
kubectl apply -f online-shop-without-footer-blue-deployment.yaml
```

### Step 8: Purane port-forwards band karein

```bash
pkill -f "kubectl port-forward"
```

### Step 9: Blue service ko dobara port-forward karein

```bash
kubectl port-forward --address 0.0.0.0 svc/online-shop-blue-deployment-service 30001:3001 -n blue-green-ns &
```

### Step 10: Result verify karein

Browser mein ye URL open karein aur page reload karein:

```
http://<Your_Instance_Public_Ip>:30001
```

Ab aapko **footer ke saath** online shop nazar aayegi (pehle footer nahi tha, ab naye feature ke taur par add ho gaya hai).

🎉 Iska matlab hai ke aap ne successfully traffic **Blue environment se Green environment** par switch kar diya.

---

## 🛠️ Troubleshooting

Agar update ke baad web app access nahi ho rahi, to terminal check karein. Shayad ye error aaya ho:

```
error: lost connection to pod
```

**Fikar ki baat nahi!** Ye is liye hota hai kyun ke cluster locally (jaise KIND mein) chal raha hai, aur deployment ke dauran jab pod replace hota hai (khaas kar `Recreate` strategy mein) to `kubectl port-forward` session toot jata hai.

🔁 **Solution:** `kubectl port-forward` command dobara chalayen:

```bash
kubectl port-forward --address 0.0.0.0 svc/online-shop-blue-deployment-service 30001:3001 -n blue-green-ns &
```

✅ Ye issue **AWS EKS, GKE ya AKS** jaise managed Kubernetes services par nahi aata, kyun ke wahan services aam tor par **NodePort, LoadBalancer ya Ingress** se expose hoti hain — `kubectl port-forward` se nahi.

---

## 🧹 Cleanup

KIND cluster delete karne ke liye:

```bash
kind delete cluster --name dep-strg
```

---

## 👤 Author

**Waqas Saleem**
DevOps Engineer

- GitHub: [Ranawaqas323421](https://github.com/Ranawaqas323421)
- Docker Hub: [waqas323421](https://hub.docker.com/u/waqas323421)
- Email: rw178722@gmail.com

---

⭐ Agar ye project helpful laga to repo ko star zaroor karein!
