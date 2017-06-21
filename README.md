#Introduccion

Microservicio encargado de manejar las transacciones para:

 * BSA
 * P2P
 * PEI
 * YPF
 
Exponse servicios para los microservicios:

 * BVTP_BuyerPaymentMethods "/private/v1/bsa/{publicRequestKey}" Se actualiza la transacción
 
Consume servicios de los microservicios:

 * BSA_Proxy_TodoPago 
 * Users
 * BVTP_BuyerPaymentMethods 
 
Se conecta a base NoSQL Couchbase y al ESB de PCI para el pago.

#Prerequisitos

 * CouchBase: Bucket BVTP - Tocar el application.properties para setear ip donde corre el container de couchbase y password para los buckets. Los puertos del container deben estar mapeados: 8091-8094:8091-8094 11210:11210.

# BVTP_Transactions
SAR: POST a `http://server:port/api/BSA/transaction`
Ejemplo de request:

    {
        "generalData": {
            "merchant": "1",
            "security": "PRISMA 86333EFD8AD0C71CEA3BF06D7BDEF90D",
            "operationDatetime": "201607040857364",
            "remoteIpAddress": "192.168.11.87",
            "channel": "BVTP"
        },
        "operationData": {
            "operationType": "Compra",
            "operationID": "1234",
            "currencyCode": "032",
            "concept": "compra",
            "amount": "999,99",
            "availablePaymentMethods": [
                "1",
                "42"
            ],
            "buyerPreselection": {
                "paymentMethodId": "42",
                "bankId": "2"
            }
        },
        "technicalData": {
            "sdk": "Java",
            "sdkversion": "2.0",
            "lenguageversion": "1.8",
            "pluginversion": "2.1",
            "ecommercename": "Bla",
            "ecommerceversion": "3.1",
            "cmsversion": "2.4"
        }
    }


Notificacion push: POST a `http://server:port/api/BSA/transaction/notificacionPush`
Ejemplo de request:

    {
        "generalData": {
            "merchant": "1",
            "security": "PRISMA 86333EFD8AD0C71CEA3BF06D7BDEF90D",
            "remoteIpAddress": "192.168.11.87",
            "publicRequestKey": "605312df-6786-4ac5-99db-974a18d364b4",
            "operationName": "Compra"
        },
        "operationData": {
            "resultCodeMedioPago": "-1",
            "resultCodeGateway": "-1",
            "idGateway": "8",
            "resultMessage": "Aprobada",
            "operationDatetime": "201607040857364",
            "ticketNumber": "7866463542424",
            "codigoAutorizacion": "455422446756567",
            "currencyCode": "032",
            "operationID": "1234",
            "concept": "compra",
            "amount": "200",
            "facilitiesPayment": "03"
        },
        "tokenizationData": {
            "publicTokenizationField": "sydguyt3e862t76ierh76487638rhkh7",
            "credentialMask": "4510XXXXX00001"
        }
    }

Consulta de transacciones: GET a `http://server:port/api/BSA/transaction/idCuenta/{idCuenta}/fechaDesde/{fechaDesde}/fechaHasta/{fechaHasta}/tipoMovimiento/{tipoMovimiento}/medioDePago/{medioDePago}/palabra/{palabra}/operationID/{operationID}/estado/{estado}/cantidadTrxPorPagina/{cantidadTrxPorPagina}/nroPagina/{nroPagina}`
Se asumira que los campos que se tengan que enviar sin valor, se enviaron como "null"
estados posibles: "Iniciada"(luego del SAR), "En Proceso" (luego del login del comprador), "Aprobada" (con un push aprobado), "Rechazada" (con un push rechazado)
tipos de movimiento posibles: "Compra", "Venta"

# Transacciones P2P 

push: POST a `http://server:port/api/transaction/p2p/push`

Ejemplo de request:

    {
        "idCuentaOrigen": "1",
        "idMpCuentaOrigen": "43",
        "idCuentaDestino": "2",
        "monto": "1700.00",
        "idEstadoP2P": "69",
        "idCanal": "7",
        "idTipoTransaccion": "10",
        "fechaAlta": "20160929125605",
        "mensaje": "mensaje",
        "codigoResultado": "1",
        "codigoResultadoResolutor": "1"
    }
 
# Transacciones PEI 

push: POST a `http://server:port/api/transaction/pei/push`

Ejemplo de request:

     {
        "cbuVendedor": "0170141440000078839908",
        "merchantId": "1",
        "usuarioVendedor": "llagrutta@baufest.com",
        "codigoAutorizacion": "123456789",
        "importe": "500,00",
        "fechaHora": "20161011122105",
        "canal": "6",
        "estado": "TX_APROBADA_PEI",
        "idMedioPago": "43",
        "tipoDocumentoComprador": "DNI",
        "codigoResultadoTP": "-1",
        "traceNumber": "90039398",
        "idCuentaVendedor": "27680",
        "fechaVencimientoTarjeta": "102019",
        "moneda": "032",
        "idBanco": "4",
        "nroDocumentoComprador": "30273649",
        "nombreMedioPago": "VISADEBITO",
        "numeroTarjeta": "4000000000000001",
        "nombreComprador": "Laura La Grutta",
        "eMailComprador": "llagrutta@yahoo.com.ar",
        "ip": "10.10.19.150",
        "alias": "LLG"
    }


## Deployment

    mvn clean package docker:build -DpushImageTag  # Push tag con version de pom.xml
    mvn clean package docker:build -DpushImageTag -DdockerImageTags=experimental  # Push tag arbitrario