---
layout: post
title: Coprac Actualización 20/09/2017
---

Configuración Nueva funcionalidades

## Sección 1

## 1)	Boton con tres puntos (...) para abrir un modal

Para poder abrir el modal se debe agregar la etiqueta Ajax.RawActionLinkToPopUp con las siguientes propiedades
        -sección de texto se debe agregar la etiqueta "<i class=\"glyphicon ico-modal\"></i>"
   -no debe tener etiqueta con “@class”
``` c#
<div class="col-md-6">
    <div class="form-group">
        @Html.LabelFor(model => model.ConceptoFinanciero.NOMBRE)
        <div class="input-group">
            @Html.TextBoxFor(model => model.ConceptoFinanciero.NOMBRE, new { id = "txtnombre", placeholder = "", @class = "form-control" })
            <div class="input-group-addon">
                @Ajax.RawActionLinkToPopUp("<i class=\"glyphicon ico-modal\"></i>", "IndexModal", "ConceptoFinanciero", null, new AjaxOptions { UpdateTargetId = "divModalConceptoFinanciero", OnBegin = "mostrarModal('Búsqueda de concepto financiero','divModalConceptoFinanciero','idModalConceptoFinanciero');", HttpMethod = "post" }, new { @id = "btnAgregarConceptoFinanciero" })
            </div>
        </div>
    </div>
</div>
```

Actualizar los archivos adjuntos(GMD.rar) en la siguiente ruta
-Content>GMD>css
-Content>GMD>img
-Content>GMD>js
*Si se registraron clases, js adicionales a la plantilla, tener presente al momento de actualizar archivo

## 2)	Ordenar desde el sistema SGA

Se deberá actualizar desde el sistema SGA como muestra la imagen
 
Se debe anteponer el numero seguido de un guion bajo ejemplo : “1_Mantenimiento”
Si no se coloca el numero el sistema ordenara según fue registrado el modulo y sub módulos
Código actualizar en el controlador Home 

``` c#
if (systemPermission != null)
{
    Login.Envio.Perfil(
        systemPermission.DefaultIfEmpty(new Permission()).First().CodigoPerfil,
        systemPermission.DefaultIfEmpty(new Permission()).First().NombrePerfil);
    Login.Envio.Sistema(
        getListSystem.DefaultIfEmpty(new SystemSGA()).First().CodigoSistem,
        getListSystem.DefaultIfEmpty(new SystemSGA()).First().NombreSistema,
        getListSystem.DefaultIfEmpty(new SystemSGA()).First().Descripcion);

    Session[Constantes.Sesion.Permisos.Lista_Modulos] = systemPermission.DistinctBy(m => new { m.CodigoModulo }).ToList()
     .Select(item => new VMPermisos.Modulo
     {
         CodigoModulo = item.CodigoModulo,
         NombreModulo = item.NombreModulo,
         Glyphicon = item.Glyphicon
     }).OrderBy(x => x.Orden).ToList();

    Session[Constantes.Sesion.Permisos.Lista_Opciones] = systemPermission.DistinctBy(p => new { p.CodigoModulo, p.CodigoPadre, p.CodigoOpcion, p.NombreOpcion, p.NombreControlador, p.NombreMetodo }).ToList()
    .Select(item => new VMPermisos.Opciones
    {
        CodigoModulo = item.CodigoModulo,
        CodigoPadre = item.CodigoPadre,
        CodigoOpcion = item.CodigoOpcion,
        NombreOpcion = item.NombreOpcion,
        NombreControlador = item.NombreControlador,
        NombreMetodo = item.NombreMetodo
    }).OrderBy(x => x.Orden).ToList();

    Session[Constantes.Sesion.Permisos.Lista_PermisosControlador] = systemPermission.DistinctBy(p => new { p.NombreControlador, p.CodigoAccion }).ToList()
    .Select(item => new VMPermisos.PermisoControlador
    {
        NombreControlador = item.NombreControlador,
        CodigoAccion = item.CodigoAccion
    }).ToList();
}
Actualizar Modelo “VMPermiso” con las siguientes líneas de código:

public class VMPermisos
{
        public class Modulo
        {
            public int CodigoModulo { get; set; }

            private string _nombreModulo;

            public string NombreModulo
            {
                get { return _nombreModulo.Split('_').Length > 1 ? _nombreModulo.Split('_')[1] : NombreModulo; }
                set { _nombreModulo = value; }
            }


            public string Glyphicon { get; set; }

            private int _orden;

            public int Orden
            {
                get { return _nombreModulo.Split('_').Length > 1 ? Convert.ToInt32(_nombreModulo.Split('_')[0]) : 0; }
                set { _orden = value; }
            }
        }

        public class Opciones
        {
            public int CodigoModulo { get; set; }

            public int CodigoPadre { get; set; }

            public int CodigoOpcion { get; set; }

            private string _nombreOpcion;

            public string NombreOpcion
            {
                get { return _nombreOpcion.Split('_').Length > 1 ? _nombreOpcion.Split('_')[1] : _nombreOpcion; }
                set { _nombreOpcion = value; }
            }


            public string NombreControlador { get; set; }

            public string NombreMetodo { get; set; }

            private int _orden;

            public int Orden
            {
                get { return _nombreOpcion.Split('_').Length > 1 ? Convert.ToInt32(_nombreOpcion.Split('_')[0]) : 0; }
                set { _orden = value; }
            }
        }

        public class PermisoControlador
        {
            public string NombreControlador { get; set; }

            public int CodigoAccion { get; set; }
        }
}
```

## 3)	Agregar página error, tiempo de sesión y re direccionar a logeo

Actualizar controlador Error con los siguientes métodos:

``` c#
public ActionResult NotFound(string aspxerrorpath)
{
    ViewData["error_path"] = aspxerrorpath;
    return PartialView();
}

public ActionResult AccessDenied()
{
    return PartialView();
}

public ActionResult SessionExpired()
{
    Session.Abandon();
    Session.Clear();
    Login.LogOut();
    return PartialView();
}
```
   Agregar vistas a cada método, las vistas se encuentran en el archivo adjunto (ErrorVistas.rar)
-Error> AccessDenied.cshtml
-Error> NotFound.cshtml
-Error> SessionExpired.cshtml
                Adicionar método a la clase login 
                               - Utilities> Login.cs
``` c#
public static void LogOut()
{
    HttpContext.Current.Session.Abandon();
}
```

## 4)	Corregir error de información de logeo 
Actualizar archivo con el archivo adjunto (Shared.rar)
                - share>_LoginPartial.cshtml 

## 5)	Mejora Notificaciones del sistema

Se realizó una mejora en las notificaciones, nuevo comportamiento:
               -No muestra mensajes duplicados
               -Mensajes de satisfacción se desvanecerá después de 5 segundos
               -Mensajes de error no se desvanecerá

Actualizar funciones del archivo 
               -share> _Layout.cshtml
*Tener en consideración aquellos script creados en esta sección fuera de la plantilla original

``` c#
<script type="text/javascript">
function GoToDefaultPage() {
    document.location.href = "@(string.Format(System.Configuration.ConfigurationManager.AppSettings["SSO_SITE_URL"], "ReturnUrl", System.Configuration.ConfigurationManager.AppSettings["CLIENT_SITE_TOKEN"]))";
}
function Notificacion(Mensaje, Tipo, Titulo, From, Aling) {
    UnloadLoading();
    var icon = "@Constantes.Controles.Alerta.Icon.Error";
    var tipo = "@Constantes.Controles.Alerta.TipoString.Error";
    var mensaje = Mensaje || "@Constantes.Controles.Alerta.Texto.MensajeDefecto";
    var titulo = Titulo || "@Constantes.Controles.Alerta.Texto.TituloDefecto";
    var from = From || "@Constantes.Controles.Alerta.From.Superior";
    var aling = Aling || "@Constantes.Controles.Alerta.Align.Derecha";

    switch (Tipo) {
    case "@Constantes.Controles.Alerta.Tipo.Success":
        icon = "@Constantes.Controles.Alerta.Icon.Success";
        tipo = "@Constantes.Controles.Alerta.TipoString.Success";
        break;
    case "@Constantes.Controles.Alerta.Tipo.Warning":
        icon = "@Constantes.Controles.Alerta.Icon.Warning";
        tipo = "@Constantes.Controles.Alerta.TipoString.Warning";
        break;
    }

    if(mensajeDuplicado(Mensaje))
        envioNotificacion(mensaje, titulo, tipo, icon, from, aling);
}

function mostrarModal(title,idmodalBody, idModal, tipo, size ) {
    if(tipo == undefined && tipo === "")
        tipo = @Constantes.Controles.Modal.Type.Informativo;
    if(size == undefined && size == "")
        size = @Constantes.Controles.Modal.Size.SizeNormal;
    instanceModal(title,idmodalBody, idModal, tipo, size);
}
function evaluarErrorActionLink(data, status, xhr) {
    if(data.Code === "@SGF.Domain.Implementation.Common.Base.Enums.Response.Exception" ) {
        instanceModal("Error del sitema", "errorModal", @Constantes.Controles.Modal.Type.Peligro);
        if(mensajeDuplicado(data.Description))
            mostrarError(data.Description);
    }
}

function evaluarResultado(data,refrescar,idModal,parametroRefrescar) {
    UnloadLoading();
    if(data.Code === "@SGF.Domain.Implementation.Common.Base.Enums.Response.Exception" ) {
        if(mensajeDuplicado(data.Description))
            Notificacion(data.Description);
    }
    else {
        if(mensajeDuplicado(data.Description))
            Notificacion(data.Description, "@Constantes.Controles.Alerta.Tipo.Success");

        if(refrescar != undefined && refrescar !== '')
            if(parametroRefrescar!=undefined && parametroRefrescar !== '')
                eval(refrescar(parametroRefrescar));
            else
                eval(refrescar());
        if (idModal != undefined && idModal !== '')
            BootstrapDialog.dialogs[idModal].close();
    }
}

function mensajeDuplicado(mensaje) {
    var flag = true;

    $("[data-notify=message]").each(function () {
        if ($(this).html() == mensaje)
            flag = false;
    });

    return flag;
}
</script>
```
## 6)	Controlador base

-Se actualizo mensaje de error 
-Escribir en ddl elmah.axd

Actualizar únicamente métodos del controlador “BaseController.cs” adjuntados en el archivo (BaseController.rar)

