<script>
  import { getLanguage } from '$lib/i18n.js';

  const common = {
    Add: 'Añadir',
    Apply: 'Aplicar',
    Back: 'Volver',
    Cancel: 'Cancelar',
    Clear: 'Limpiar',
    Close: 'Cerrar',
    Create: 'Crear',
    Delete: 'Eliminar',
    Edit: 'Editar',
    Export: 'Exportar',
    Filter: 'Filtrar',
    Filters: 'Filtros',
    Help: 'Ayuda',
    'Loading...': 'Cargando...',
    Loading: 'Cargando',
    New: 'Nuevo',
    Next: 'Siguiente',
    'No results': 'Sin resultados',
    'No data': 'Sin datos',
    Open: 'Abrir',
    Previous: 'Anterior',
    Refresh: 'Actualizar',
    Reset: 'Restablecer',
    Save: 'Guardar',
    Search: 'Buscar',
    Select: 'Seleccionar',
    Send: 'Enviar',
    Settings: 'Configuración',
    Submit: 'Enviar',
    Update: 'Actualizar',
    View: 'Ver',
    Yes: 'Sí',
    No: 'No',
    Today: 'Hoy',
    Yesterday: 'Ayer',
    Tomorrow: 'Mañana',
    'Created at': 'Creado el',
    'Updated at': 'Actualizado el',
    'Created by': 'Creado por',
    'Assigned to': 'Asignado a',
    Status: 'Estado',
    Name: 'Nombre',
    Email: 'Correo electrónico',
    Phone: 'Teléfono',
    Address: 'Dirección',
    Description: 'Descripción',
    Source: 'Fuente',
    Company: 'Empresa',
    Contact: 'Contacto',
    Account: 'Cuenta',
    Lead: 'Prospecto',
    Opportunity: 'Oportunidad',
    Task: 'Tarea',
    Ticket: 'Ticket',
    Invoice: 'Factura',
    Product: 'Producto',
    Team: 'Equipo',
    Active: 'Activo',
    Inactive: 'Inactivo',
    Pending: 'Pendiente',
    Completed: 'Completado',
    Closed: 'Cerrado',
    'In progress': 'En progreso',
    'Are you sure?': '¿Estás seguro?',
    'Something went wrong': 'Algo salió mal',
    'Something went wrong. Please try again.': 'Algo salió mal. Inténtalo de nuevo.',
    'No items found': 'No se encontraron elementos',
    'No records found': 'No se encontraron registros',
    'Try again': 'Intentar de nuevo'
    , 'New invoice': 'Nueva factura'
    , 'Go to pipeline': 'Ir al pipeline'
    , 'New estimate': 'Nueva cotización'
    , 'Start from a deal': 'Comenzar desde un negocio'
    , 'New product': 'Nuevo producto'
    , 'Ask an admin to add products.': 'Pide a un administrador que añada productos.'
    , Retired: 'Retirado'
    , 'Admins only': 'Solo administradores'
    , 'Back to products': 'Volver a productos'
    , 'List price': 'Precio de lista'
    , Currency: 'Moneda'
    , Category: 'Categoría'
    , Availability: 'Disponibilidad'
    , 'Sellable, appears in the line-item picker': 'Vendible, aparece al añadir líneas'
    , 'Retired, kept for history, hidden from the picker': 'Retirado, se conserva en el historial y se oculta'
    , 'Add product': 'Añadir producto'
    , 'Save changes': 'Guardar cambios'
    , 'Delete this product for good?': '¿Eliminar este producto definitivamente?'
    , 'Keep it': 'Conservarlo'
    , 'New schedule': 'Nuevo calendario'
    , Schedule: 'Calendario'
    , Every: 'Cada'
    , 'When it generates': 'Cuándo se genera'
    , 'Sends automatically': 'Se envía automáticamente'
    , 'Drafts, waits for you': 'Borrador, espera tu revisión'
    , 'Who and what': 'Quién y qué'
    , 'Choose an account': 'Selecciona una cuenta'
    , Title: 'Título'
    , Cadence: 'Frecuencia'
    , Frequency: 'Periodicidad'
    , 'Every N days': 'Cada N días'
    , 'Start date': 'Fecha de inicio'
    , 'Next generation date': 'Próxima fecha de generación'
    , 'End date (optional)': 'Fecha de finalización (opcional)'
    , 'Payment terms': 'Condiciones de pago'
    , 'off leaves a draft for you to review and send': 'desactivado deja un borrador para revisar y enviar'
    , Lines: 'Líneas'
    , 'Add from catalogue…': 'Añadir del catálogo…'
    , Qty: 'Cantidad'
    , 'Unit price': 'Precio unitario'
    , Adjustments: 'Ajustes'
    , Discount: 'Descuento'
    , None: 'Ninguno'
    , Percentage: 'Porcentaje'
    , 'Fixed amount': 'Importe fijo'
    , 'Tax rate %': 'Tasa de impuesto %'
    , Estimate: 'Cotización'
    , Billed: 'Facturado'
    , Amount: 'Importe'
    , Valid: 'Válida'
    , Due: 'Vencimiento'
    , Age: 'Antigüedad'
    , Next: 'Siguiente'
  };

  const originalText = new WeakMap();
  const originalAttributes = new WeakMap();

  function translate(language) {
    const walker = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT);
    let node;
    while ((node = walker.nextNode())) {
      if (!node.parentElement || ['SCRIPT', 'STYLE', 'TEXTAREA'].includes(node.parentElement.tagName)) continue;
      const original = originalText.get(node) ?? node.nodeValue;
      originalText.set(node, original);
      const key = original.trim();
      const next = language === 'es' && common[key] ? original.replace(key, common[key]) : original;
      if (node.nodeValue !== next) node.nodeValue = next;
    }

    document.querySelectorAll('input, textarea, [aria-label], [title]').forEach((element) => {
      const attrs = originalAttributes.get(element) || {};
      for (const name of ['placeholder', 'aria-label', 'title']) {
        if (!element.hasAttribute(name)) continue;
        attrs[name] ??= element.getAttribute(name);
        const value = attrs[name];
        const next = language === 'es' && common[value] ? common[value] : value;
        if (element.getAttribute(name) !== next) element.setAttribute(name, next);
      }
      originalAttributes.set(element, attrs);
    });
  }

  $effect(() => {
    const run = () => translate(getLanguage());
    run();
    const observer = new MutationObserver(run);
    observer.observe(document.body, { childList: true, subtree: true, characterData: true });
    window.addEventListener('bottlecrm-language', run);
    return () => {
      observer.disconnect();
      window.removeEventListener('bottlecrm-language', run);
    };
  });
</script>