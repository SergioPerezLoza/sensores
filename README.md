# sensores
## Sergio Pérez Lozano
## Alu0101473260@ull.edu.es
### 1. Mostrar Valores en la UI

Se creó un sistema de interfaz en Unity que incluye varios elementos de texto dentro de un Canvas. Cada elemento muestra los valores capturados en tiempo real por los sensores del dispositivo:

Velocidad angular: Utilizando el giroscopio (Input.gyro).

Aceleración: Capturada con el acelerómetro (Input.acceleration).

Altitud, longitud y latitud: Obtenidas a través del GPS (Input.location.lastData).
```csharp
using UnityEngine;
using UnityEngine.UI;

public class SensorDisplay : MonoBehaviour
{
    // Referencias a los textos de la UI
    public Text angularVelocityText;
    public Text accelerationText;
    public Text altitudeText;
    public Text gravityText;
    public Text latitudeText;
    public Text longitudeText;

    private void Start()
    {
        // Inicia los servicios necesarios
        Input.location.Start(10, 1); // Servicio de localización
        Input.compass.enabled = true; // Activa la brújula
    }

    private void Update()
    {
        // Velocidad Angular (Ejemplo con giroscopio si está disponible)
        if (SystemInfo.supportsGyroscope)
        {
            Vector3 angularVelocity = Input.gyro.rotationRate;
            angularVelocityText.text = $"Angular Velocity: {angularVelocity}";
        }
        else
        {
            angularVelocityText.text = "Angular Velocity: Not supported";
        }

        // Aceleración
        Vector3 acceleration = Input.acceleration;
        accelerationText.text = $"Acceleration: {acceleration}";

        // Altitud, Latitud y Longitud
        if (Input.location.status == LocationServiceStatus.Running)
        {
            var locationData = Input.location.lastData;
            latitudeText.text = $"Latitude: {locationData.latitude}";
            longitudeText.text = $"Longitude: {locationData.longitude}";
            altitudeText.text = $"Altitude: {locationData.altitude}";
        }
        else
        {
            latitudeText.text = "Latitude: Waiting for GPS";
            longitudeText.text = "Longitude: Waiting for GPS";
            altitudeText.text = "Altitude: Waiting for GPS";
        }

        // Gravedad (puede usarse el acelerómetro para estimar gravedad)
        Vector3 gravity = Physics.gravity;
        gravityText.text = $"Gravity: {gravity}";
    }

    private void OnDestroy()
    {
        // Detiene los servicios al salir
        Input.location.Stop();
    }
}

'''

### 2. Movimiento del Samurái

Un modelo de samurái fue configurado para reaccionar según los datos del dispositivo:

Orientación hacia el norte: Usando la brújula (Input.compass.trueHeading), el samurái rota continuamente hacia el norte.

Movimiento con el acelerómetro: El samurái avanza o retrocede según la inclinación del dispositivo (valores negativos del eje Z del acelerómetro).

Límites de movimiento: Si el dispositivo está fuera de un rango de coordenadas (especificado en el script), el samurái se detiene.

Rotaciones suaves: Implementadas con interpolación Quaternion.Slerp para asegurar transiciones naturales entre orientaciones.


```csharp
using UnityEngine;

public class SamuraiController : MonoBehaviour
{
    public float smoothing = 0.1f; // Suavidad de rotación
    public float speedMultiplier = 5f; // Factor de velocidad
    public float minLatitude = -90f; // Latitud mínima permitida
    public float maxLatitude = 90f; // Latitud máxima permitida
    public float minLongitude = -180f; // Longitud mínima permitida
    public float maxLongitude = 180f; // Longitud máxima permitida

    private Quaternion targetRotation;

    private void Start()
    {
        // Inicia el servicio de localización y habilita la brújula
        Input.location.Start(10, 1);
        Input.compass.enabled = true;
    }

    private void Update()
    {
        // Orientar al samurái hacia el norte
        float heading = Input.compass.trueHeading; // Dirección hacia el norte
        targetRotation = Quaternion.Euler(0, heading, 0); // Rotación hacia el norte
        transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, smoothing);

        // Movimiento basado en la aceleración del dispositivo
        Vector3 acceleration = Input.acceleration;
        float forwardSpeed = -acceleration.z * speedMultiplier; // Invertimos Z
        transform.Translate(Vector3.forward * forwardSpeed * Time.deltaTime);

        // Verificar los límites de latitud y longitud
        if (Input.location.status == LocationServiceStatus.Running)
        {
            var locationData = Input.location.lastData;
            if (locationData.latitude < minLatitude || locationData.latitude > maxLatitude ||
                locationData.longitude < minLongitude || locationData.longitude > maxLongitude)
            {
                // Detenemos al samurái
                StopMovement();
            }
        }
    }

    private void StopMovement()
    {
        // Detenemos el movimiento estableciendo velocidad a 0
        transform.Translate(Vector3.zero);
    }

    private void OnDestroy()
    {
        // Detenemos el servicio de localización al cerrar
        Input.location.Stop();
    }
}

```
