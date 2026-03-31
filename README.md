# Criando Aplicativo Device Owner no Android

Este projeto é um exemplo prático de como criar uma aplicação Android com privilégios de **Device Owner**, utilizando a API `DevicePolicyManager` para controlar funcionalidades do dispositivo — neste caso, bloqueando o acesso do usuário às configurações de Wi-Fi.

A API `DevicePolicyManager`, disponível desde o Android 5.0, permite que apps gerenciem recursos do sistema como desabilitar câmera, impor políticas de segurança e controlar diversas funcionalidades do dispositivo.

---

## Pré-requisito importante

> Para que o app funcione corretamente como Device Owner, **o dispositivo deve estar sem contas configuradas e sem chip da operadora**. Em emuladores, utilize um AVD sem conta Google cadastrada.

---

## Componentes do Projeto

### 1. DeviceOwnerReceiver

Classe que herda de `DeviceAdminReceiver` para comunicação com o sistema operacional. O método `onProfileProvisioningComplete` é chamado quando o provisionamento do dispositivo é concluído.

```kotlin
class DeviceOwnerReceiver : DeviceAdminReceiver() {
    override fun onProfileProvisioningComplete(context: Context, intent: Intent) {
        val manager = context.getSystemService(DEVICE_POLICY_SERVICE)
            as DevicePolicyManager
        val componentName = ComponentName(
            context.applicationContext,
            DeviceOwnerReceiver::class.java
        )
        manager.setProfileName(componentName, DEVICE_OWNER_NAME)
        val intent = Intent(context, MainActivity::class.java)
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
        context.startActivity(intent)
    }

    companion object {
        const val DEVICE_OWNER_NAME = "Device Owner"
    }
}
```

---

### 2. Arquivo de Políticas XML (`device_owner_rules.xml`)

Define as permissões administrativas solicitadas pelo app:

```xml
<?xml version="1.0" encoding="utf-8"?>
<device-admin>
    <uses-policies>
        <limit-password/>
        <watch-login/>
        <reset-password/>
        <force-lock/>
        <wipe-data/>
        <expire-password/>
        <encrypted-storage/>
        <disable-camera/>
    </uses-policies>
</device-admin>
```

---

### 3. AndroidManifest.xml

Registra o `DeviceOwnerReceiver` como um receiver do sistema e declara a permissão de Wi-Fi:

```xml
<uses-permission android:name="android.permission.MANAGE_DEVICE_POLICY_WIFI" />

<receiver
    android:name=".DeviceOwnerReceiver"
    android:description="@string/app_name"
    android:label="@string/app_name"
    android:permission="android.permission.BIND_DEVICE_ADMIN"
    android:exported="true">
    <meta-data
        android:name="android.app.device_admin"
        android:resource="@xml/device_owner_rules" />
    <intent-filter>
        <action android:name="android.app.action.PROFILE_PROVISIONING_COMPLETE" />
    </intent-filter>
</receiver>
```

---

### 4. MainActivity — Aplicando a Restrição de Wi-Fi

Instancie os objetos necessários:

```kotlin
val dpm = getSystemService(DEVICE_POLICY_SERVICE) as DevicePolicyManager
val componentName = ComponentName(this, DeviceOwnerReceiver::class.java)
```

Aplicar a restrição (impede o usuário de configurar Wi-Fi):

```kotlin
dpm.addUserRestriction(componentName, UserManager.DISALLOW_CONFIG_WIFI)
```

Remover a restrição:

```kotlin
dpm.clearUserRestriction(componentName, UserManager.DISALLOW_CONFIG_WIFI)
```

---

## Registrando o App como Device Owner via ADB

**Passo 1:** Navegue até o diretório das ferramentas do Android SDK:

```bash
cd C:\Users\SEU_USUARIO\AppData\Local\Android\Sdk\platform-tools
```

**Passo 2:** Execute o comando para definir o app como Device Owner:

```bash
./adb shell dpm set-device-owner com.example.devicepolicymanage/.DeviceOwnerReceiver
```

Uma mensagem de sucesso confirma que o app foi registrado como Device Owner.

### Entendendo o comando

| Parte | Descrição |
|---|---|
| `./adb` | Executa a ferramenta ADB do diretório atual |
| `shell` | Abre um shell remoto no dispositivo |
| `dpm` | Device Policy Manager via linha de comando |
| `set-device-owner` | Define o app como Device Owner |
| `com.example.devicepolicymanage/.DeviceOwnerReceiver` | Identificação do aplicativo e receiver |

---

## Removendo os Privilégios Administrativos

Via ADB:

```bash
./adb shell dpm remove-active-admin com.example.devicepolicymanage/.DeviceOwnerReceiver
```

Via código (antes de desinstalar o app):

```kotlin
dpm.clearDeviceOwnerApp(packageName)
```

---

## Referências

- [DevicePolicyManager — Documentação oficial Android](https://developer.android.com/reference/android/app/admin/DevicePolicyManager)
- [Android Device Admin](https://developer.android.com/work/device-admin)
- [Artigo original — Criando Aplicativo Device Owner no Android](https://medium.com/@mateus.oliver_/criando-aplicativo-device-owner-no-android-f222832c75ed)
