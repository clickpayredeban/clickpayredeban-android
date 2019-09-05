# clickpayredeban-android
El SDK Android de Click Pay Redeban es una biblioteca que permite a los desarrolladores conectarse fácilmente al API Click Pay Redeban de Tarjetas de Crédito.


![Example](https://developers.clickpayredeban.com/wp-content/uploads/2017/10/Screenshot_20171025-232311-e1509058423311.png)


## Instalación

### Android Studio (o Gradle)

Agrege esta línea en su aplicación  `build.gradle` dentro de la sección `dependencies`:


    compile 'com.clickpayredeban:clickpayredeban-android:1.2.8'

### ProGuard

Si usted está planeando optimiar su aplicación con ProGuard, asegúrese de excluir los enlaces de Click Pay Redeban. Ustede puede realizarlo añadiendo lo siguiente al archivo `proguard.cfg`  de su app:

    -keep class com.clickpayredeban.android.** { *; }

## Uso

### Utilizando el CardMultilineWidget

Puede agregar un widget a sus aplicaciones que maneje fácilmente los estados de la interfaz de usuario para recopilar datos de la tarjeta.

Primero, añada el CardMultilineWidget a su layout.


```xml
<com.clickpayredeban.android.view.CardMultilineWidget
        android:id="@+id/card_multiline_widget"
        android:layout_alignParentTop="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
```

Ustede puede personalizar la vista con las siguientes etiquetas:

```xml
app:shouldShowPostalCode="true"
app:shouldShowPaymentezLogo="true"
app:shouldShowCardHolderName="true"
app:shouldShowScanCard="true"
```

Para usar cualquiera de estas etiquetas, deberá habilitar el espacio de nombres XML de la aplicación en algún lugar del layout.

```xml
xmlns:app="http://schemas.android.com/apk/res-auto"
```

Para obtener un objeto `Card` del` CardMultilineWidget`, pidale al widget su tarjeta.

```java
Card cardToSave = cardWidget.getCard();
if (cardToSave == null) {
    Alert.show(mContext,
        "Error",
        "Invalid Card Data");
    return;
}
```
Si la `Card` devuelta es null , se mostrarán estados de error en los campos que deben corregirse.

Una vez que tenga un objeto `Card` no null regresado desde el widget, puede llamar a [addCard](#addCard).


### Inicializar Biblioteca

Usted debe inicializar la biblioteca en su Aplicación en su primera actividad.
You should initialize the library on your Application or in your first Activity. 


```java
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import com.paymentez.android.Paymentez;
import com.paymentez.examplestore.utils.Constants;

public class MainActivity extends ActionBarActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);      
      setContentView(R.layout.activity_main);
      
      /**
       * Init library
       *
       * @param test_mode false to use production environment
       * @param paymentez_client_app_code provided by Paymentez.
       * @param paymentez_client_app_key provided by Paymentez.
       */
      Paymentez.setEnvironment(Constants.PAYMENTEZ_IS_TEST_MODE, Constants.PAYMENTEZ_CLIENT_APP_CODE, Constants.PAYMENTEZ_CLIENT_APP_KEY);
      
      
       // In case you have your own Fraud Risk Merchant Id
       //Paymentez.setRiskMerchantId(1000);
       // Note: for most of the devs, that's not necessary.
    }
}
```

### addCard

addCard convierte datos confidenciales de la tarjeta en un token de un solo uso que puede pasar de forma segura a su servidor para realizar el cobro al usuario.


```java
Paymentez.addCard(mContext, uid, email, cardToSave, new TokenCallback() {

    public void onSuccess(Card card) {
        
        if(card != null){
            if(card.getStatus().equals("valid")){
                Alert.show(mContext,
                        "Card Successfully Added",
                        "status: " + card.getStatus() + "\n" +
                                "Card Token: " + card.getToken() + "\n" +
                                "transaction_reference: " + card.getTransactionReference());

            } else if (card.getStatus().equals("review")) {
                Alert.show(mContext,
                        "Card Under Review",
                        "status: " + card.getStatus() + "\n" +
                                "Card Token: " + card.getToken() + "\n" +
                                "transaction_reference: " + card.getTransactionReference());

            } else {
                Alert.show(mContext,
                        "Error",
                        "status: " + card.getStatus() + "\n" +
                                "message: " + card.getMessage());
            }


        }

        //TODO: Create charge or Save Token to your backend
    }

    public void onError(PaymentezError error) {        
        Alert.show(mContext,
                "Error",
                "Type: " + error.getType() + "\n" +
                        "Help: " + error.getHelp() + "\n" +
                        "Description: " + error.getDescription());

        //TODO: Handle error
    }

});
```
El primer argumento del addCard es mContext (Context).
+ mContext. Context de la Actividad actual.

El segundo argumento del addCard es uid (Cadena).
+ uid Identificador de comprador. Este es el identificador que usa dentro de su aplicación; lo recibirá en las notificaciones.

El tercer argumento del addCard es el email (Cadena).
+ email Email del comprador

El cuarto argumento del addCard es un objeto de tipo Card. Un objeto Card contiene los siguientes campos:

+ number: número de tarjeta como cadena sin ningún separador, por ejemplo '4242424242424242'.
+ holderName: nombre del tarjehabiente.
+ expMonth: entero que representa el mes de expiración de la tarjeta, por ejemplo 12.
+ expYear: entero que represetna el año de expiración de la tarjeta, por ejemplo 2019.
+ cvc: código de seguridad de la tarjeta, como cadena, por ejemplo '123'.
+ type: el tipo de tarjeta.

El quinto argumento tokenCallBack es un callback que usted provee para manejar las respuestas recibidas de ClickPayRedeban.


The fifth argument tokenCallback is a callback you provide to handle responses from Paymentez.
Deberá enviar el token a su servidor para procesar onSuccess y notificar al usuario onError.

Aquí se muestra un ejemplo de implementación de callback del token:
```java
Paymentez.addCard(
    mContext, uid, email, cardToSave,
    new TokenCallback() {
        public void onSuccess(Card card) {
            // Send token to your own web service
            MyServer.chargeToken(card.getToken());
        }
        public void onError(PaymentezError error) {
            Toast.makeText(getContext(),
                error.getDescription(),
                Toast.LENGTH_LONG).show();
        }
    }
);
```

`addCard` es una llamada asíncrona - regresa inmediatamente e invoca el callback en el hilo de la interfaz, cuando recibe una respuesta de los servidores de Click Pay Redeban.


### getSessionId

El Session ID es un parámetro que Click Pay Redeban utiliza para fines de antifraude.
Llave esté método cuando quiera recabar la información del dispositivo.

```java
String session_id = Paymentez.getSessionId(mContext);
```
Una vez que tenga el Session ID, usted podrá pasarlo a su servidor para realizar el cobro a su usuario.


### Herramientas de validación del lado del Cliente
El objeto Card permite validar la información capturada por el usuario antes de enviarla a Click Pay Redeban.


#### validateNumber

Verifica que el número tenga el formato correcto y pase el algoritmo de [Luhn](http://en.wikipedia.org/wiki/Luhn_algorithm).

#### validateExpiryDate

Comprueba que la fecha de expiración representa una fecha real futura.

#### validateCVC

Comprueba si el número proporcionado podría ser un código de verificación válido o no.

#### validateCard

Método que valida el número de tarjeta, la fecha de expiración y el CVC.

## Applicaciones de ejemplo

En el repositorio hay una applicación de ejemplo incluida:

- PaymentezStore es  un recorrido completo de la construcción de una tienda, incluida la conexión a un back end. 

https://cdn.paymentez.com/apps/paymentez-example-1-2-7.apk

Para construir y ejecutar la aplicación de ejemplo, clone el repositorio y abra el proyecto.

### Introducción a la aplicación de ejemplo de Android 

Nota: la aplicación requiere [Android SDK](https://developer.android.com/studio/index.html) y [Gradle](https://gradle.org/) para coinstruir y ejecutar.


### Construyendo y Ejecutando la PaymentStore 

Antes de que pueda pueda correr la aplicación PaymentStore, ested necesita proveerla con las credenciales de Click Pay Redeban y un backend de muestra.

1. Si aún no cuenta con credenciales, pídalas a su contancto en el equipo de Click Pay Redeban.
2. Dirígase a https://github.com/clickpayredeban/example-java-backend y haga click en "Deploy to Heroku" (es posible que tengas que registrarte para obtener una cuenta de Heroku como parte de este proceso). Proporcione sus credenciales de servidor de ClickPayRedeban `SERVER_APP_CODE` y` SERVER_APP_KEY` en los campos "Env". Haga click en "Deloy for Free".
3. Abra el proyecto en Android Studio.
4. Reemplace las constantes `PAYMENTEZ_CLIENT_APP_CODE` and `PAYMENTEZ_CLIENT_APP_KEY` en Constants.java con sus propias credenciales de Click Pay Redeban.
5. Reemplace la variable `BACKEND_URL` en el archivo Constants.java con la URL de la aplicación que Heroku le proporciona (por ejemplo," https://my-example-app.herokuapp.com ")
6. Ejecute el proyecto.

Nota importante: si solo tiene un APP_CODE, suponga que es su `SERVER_APP_CODE`. Por lo tanto, debe solicitar a su contacto en el equipo de Click Pay Redeban su `CLIENT_APP_CODE`.

