import React, { useState, useEffect, useRef } from 'react';
import { 
  Car, Palette, FileText, Calculator, Save, Copy, Image, Settings, 
  Trash2, Search, Download, Upload, Camera, Clock, BarChart3, 
  Package, Bell, Eye, Users, MapPin, Star, Filter, Calendar,
  DollarSign, Wrench, Zap, Activity, Target, TrendingUp, Plus, 
  Check, ChevronDown, ChevronRight, Share2, Printer, Mail, Link,
  LayoutDashboard, History, Cloud, Server, Database, Cpu, Box
} from 'lucide-react';

// Componentes
import VehicleTab from './components/VehicleTab';
import CustomizationTab from './components/CustomizationTab';
import CostsTab from './components/CostsTab';
import PromptsTab from './components/PromptsTab';
import ProjectTab from './components/ProjectTab';
import SavedProjectsTab from './components/SavedProjectsTab';
import StatsDashboard from './components/StatsDashboard';
import NotificationCenter from './components/NotificationCenter';
import TeamCollaboration from './components/TeamCollaboration';
import CarPreview3D from './components/CarPreview3D';
import ModificationSelector from './components/ModificationSelector';
import ColorPicker from './components/ColorPicker';

// Servicios
import { generateCarImage } from './services/aiService';
import { fetchVehicleData } from './services/vehicleApiService';

const AutoDesignStudio = () => {
  // Estados principales
  const [activeTab, setActiveTab] = useState('vehiculo');
  const [vehicleData, setVehicleData] = useState({
    marca: '',
    modelo: '',
    año: new Date().getFullYear().toString(),
    tipo: '',
    color_original: '#CCCCCC',
    vin: '',
    kilometraje: '0',
    propietario: '',
    ubicacion: '',
    motor: '',
    transmision: '',
    estado_general: 'Bueno'
  });
  
  const [customization, setCustomization] = useState({
    color_principal: '#FF0000',
    color_secundario: '',
    acabado: 'Brillante',
    tipo_pintura: 'Metálico',
    detalles_especiales: '',
    fascias: '',
    accesorios: '',
    estilo: 'Deportivo',
    prioridad: 'Media',
    dificultad: 'Media',
    modificaciones: []
  });

  const [projectData, setProjectData] = useState({
    cliente: '',
    contacto: '',
    email: '',
    fecha: new Date().toISOString().split('T')[0],
    precio_estimado: '',
    materiales: '',
    tiempo_estimado: '15',
    notas: '',
    estado: 'Presupuesto',
    fecha_inicio: '',
    fecha_entrega: '',
    progreso: 0,
    equipo: []
  });

  const [generatedPrompts, setGeneratedPrompts] = useState({
    frontal: '',
    lateral: '',
    trasero: '',
    superior: '',
    interior: '',
    detalle: ''
  });

  const [savedProjects, setSavedProjects] = useState(() => {
    const saved = localStorage.getItem('carProjects');
    return saved ? JSON.parse(saved) : [];
  });

  const [inventory, setInventory] = useState(() => {
    const saved = localStorage.getItem('carInventory');
    return saved ? JSON.parse(saved) : [];
  });

  const [reminders, setReminders] = useState(() => {
    const saved = localStorage.getItem('carReminders');
    return saved ? JSON.parse(saved) : [];
  });

  const [projectImages, setProjectImages] = useState({});
  const [searchTerm, setSearchTerm] = useState('');
  const [notifications, setNotifications] = useState([]);
  const [filterStatus, setFilterStatus] = useState('all');
  const [compareProjects, setCompareProjects] = useState([]);
  const [showStats, setShowStats] = useState(false);
  const [darkMode, setDarkMode] = useState(false);
  const [costBreakdown, setCostBreakdown] = useState({});
  const [showColorPicker, setShowColorPicker] = useState(false);
  const [selectedModification, setSelectedModification] = useState('');
  const [teamMembers] = useState([
    { id: 1, name: 'Juan Pérez', role: 'Diseñador', skills: ['Photoshop', '3D Modeling'] },
    { id: 2, name: 'María García', role: 'Pintor', skills: ['Pintura', 'Preparación'] },
    { id: 3, name: 'Carlos López', role: 'Mecánico', skills: ['Motores', 'Suspensión'] },
    { id: 4, name: 'Ana Martínez', role: 'Project Manager', skills: ['Planificación', 'Coordinación'] }
  ]);
  
  const [projectHistory, setProjectHistory] = useState([]);
  const [currentProjectId, setCurrentProjectId] = useState(null);
  const [viewAngle, setViewAngle] = useState('front');

  // Tipos de datos
  const tiposVehiculo = [
    'Sedan', 'Hatchback', 'SUV', 'Pickup', 'Coupe', 'Convertible', 
    'Wagon', 'Van', 'Truck', 'Deportivo', 'Clásico', 'Motocicleta',
    'ATV', 'Trailer', 'Bus', 'Camión Pesado'
  ];

  const tiposPintura = [
    { value: 'solido-mate', name: 'Sólido mate', materialMultiplier: 1.0, laborMultiplier: 1.0 },
    { value: 'solido-brillante', name: 'Sólido brillante', materialMultiplier: 1.2, laborMultiplier: 1.1 },
    { value: 'metalico', name: 'Metálico', materialMultiplier: 1.5, laborMultiplier: 1.3 },
    { value: 'perlado', name: 'Perlado', materialMultiplier: 1.8, laborMultiplier: 1.5 },
    { value: 'candy', name: 'Candy', materialMultiplier: 2.0, laborMultiplier: 1.8 },
    { value: 'chamaleon', name: 'Chamaleon', materialMultiplier: 2.5, laborMultiplier: 2.2 },
    { value: 'cromado', name: 'Cromado', materialMultiplier: 3.0, laborMultiplier: 2.5 },
    { value: 'vinilo', name: 'Vinilo', materialMultiplier: 1.7, laborMultiplier: 1.6 },
    { value: 'wrap-completo', name: 'Wrap completo', materialMultiplier: 2.2, laborMultiplier: 2.0 }
  ];

  const estilosCustom = [
    'Clásico', 'Deportivo', 'Racing', 'Luxury', 'Muscle Car', 
    'JDM', 'Euro Style', 'Low Rider', 'Off Road', 'Tuning', 'Drift',
    'Retro', 'Futurista', 'Minimalista', 'Agresivo', 'Elegante'
  ];

  const estadosProyecto = [
    'Presupuesto', 'Aprobado', 'En proceso', 'Pausado',
    'Finalizado', 'Entregado', 'Cancelado', 'Revision'
  ];

  const coloresPopulares = [
    '#FF0000', '#00FF00', '#0000FF', '#FFFF00', '#FF00FF', '#00FFFF',
    '#000000', '#FFFFFF', '#808080', '#800000', '#008000', '#000080',
    '#FFA500', '#800080', '#FFC0CB', '#A52A2A', '#8B0000', '#4B0082',
    '#FFD700', '#C0C0C0', '#36454F', '#2F4F4F', '#708090', '#B22222'
  ];

  const modificationsData = [
    { name: 'Alerón trasero', cost: 250, difficulty: 'Media', time: '4 horas' },
    { name: 'Llantas deportivas', cost: 1200, difficulty: 'Baja', time: '2 horas' },
    { name: 'Sistema de escape', cost: 800, difficulty: 'Alta', time: '6 horas' },
    { name: 'Suspensión ajustable', cost: 1500, difficulty: 'Alta', time: '8 horas' },
    { name: 'Calcomanías personalizadas', cost: 300, difficulty: 'Baja', time: '3 horas' },
    { name: 'Luces LED', cost: 450, difficulty: 'Media', time: '5 horas' },
    { name: 'Interior piel deportivo', cost: 2200, difficulty: 'Alta', time: '20 horas' },
    { name: 'Sistema de audio', cost: 1800, difficulty: 'Media', time: '10 horas' },
    { name: 'Frenos mejorados', cost: 1200, difficulty: 'Alta', time: '8 horas' },
    { name: 'Turbocompresor', cost: 3500, difficulty: 'Alta', time: '15 horas' },
    { name: 'Intercooler', cost: 900, difficulty: 'Media', time: '6 horas' },
    { name: 'Kit body wide', cost: 2800, difficulty: 'Alta', time: '25 horas' },
    { name: 'Ventanillas polarizadas', cost: 600, difficulty: 'Baja', time: '3 horas' }
  ];

  const complexityLevels = [
    { value: 'Baja', multiplier: 0.8 },
    { value: 'Media', multiplier: 1.0 },
    { value: 'Alta', multiplier: 1.3 }
  ];

  // Efectos para persistencia
  useEffect(() => {
    localStorage.setItem('carProjects', JSON.stringify(savedProjects));
  }, [savedProjects]);

  useEffect(() => {
    localStorage.setItem('carInventory', JSON.stringify(inventory));
  }, [inventory]);

  useEffect(() => {
    localStorage.setItem('carReminders', JSON.stringify(reminders));
  }, [reminders]);

  useEffect(() => {
    calculateCostBreakdown();
  }, [customization, vehicleData]);

  // Guardar versión actual del proyecto
  const saveVersion = () => {
    const version = {
      timestamp: new Date(),
      vehicleData: {...vehicleData},
      customization: {...customization},
      projectData: {...projectData}
    };
    
    setProjectHistory(prev => [...prev, version]);
    addNotification('Versión del proyecto guardada', 'success');
  };

  // Recuperar versión anterior
  const restoreVersion = (versionIndex) => {
    const version = projectHistory[versionIndex];
    setVehicleData({...version.vehicleData});
    setCustomization({...version.customization});
    setProjectData({...version.projectData});
    addNotification('Versión restaurada exitosamente', 'success');
  };

  // Función para calcular costos automáticamente
  const calculateCostBreakdown = () => {
    const vehicleSize = getVehicleSize(vehicleData.tipo);
    const paintType = tiposPintura.find(p => p.name === customization.tipo_pintura);
    const complexity = complexityLevels.find(c => c.value === customization.dificultad);
    
    if (!paintType || !complexity) return;
    
    const materialCost = vehicleSize * paintType.materialMultiplier;
    const laborCost = vehicleSize * complexity.multiplier * paintType.laborMultiplier;
    
    // Costo de modificaciones
    const modCost = customization.modificaciones.reduce((acc, modName) => {
      const mod = modificationsData.find(m => m.name === modName);
      return acc + (mod?.cost || 0);
    }, 0);
    
    setCostBreakdown({
      materiales: materialCost,
      manoObra: laborCost,
      modificaciones: modCost,
      total: materialCost + laborCost + modCost,
      tiempo: Math.ceil((materialCost + laborCost) / 100) + ' días'
    });
    
    // Actualizar precio estimado en proyecto
    setProjectData(prev => ({
      ...prev,
      precio_estimado: (materialCost + laborCost + modCost).toFixed(2)
    }));
  };

  const getVehicleSize = (tipo) => {
    const sizes = {
      'Motocicleta': 200, 'Coupe': 400, 'Sedan': 500, 'Hatchback': 450,
      'SUV': 600, 'Pickup': 650, 'Van': 700, 'Truck': 800, 'Bus': 1000
    };
    return sizes[tipo] || 500;
  };

  // Función para agregar modificación
  const addModification = (mod) => {
    if (mod && !customization.modificaciones.includes(mod)) {
      setCustomization(prev => ({
        ...prev,
        modificaciones: [...prev.modificaciones, mod]
      }));
    }
  };

  // Función para eliminar modificación
  const removeModification = (mod) => {
    setCustomization(prev => ({
      ...prev,
      modificaciones: prev.modificaciones.filter(m => m !== mod)
    }));
  };

  // Función para generar prompts mejorados
  const generatePrompts = () => {
    if (!vehicleData.marca || !vehicleData.modelo) {
      addNotification('Complete los datos del vehículo primero', 'error');
      return;
    }

    const baseDescription = `${vehicleData.año} ${vehicleData.marca} ${vehicleData.modelo} ${vehicleData.tipo}`;
    const colorDescription = customization.color_secundario 
      ? `painted in ${customization.color_principal} with ${customization.color_secundario} accents`
      : `painted in ${customization.color_principal}`;
    
    const finishDescription = `${customization.acabado} ${customization.tipo_pintura} finish`;
    const styleDescription = customization.estilo ? `${customization.estilo} style` : '';
    const detailsDescription = customization.detalles_especiales ? `, ${customization.detalles_especiales}` : '';
    const accesoriosDescription = customization.accesorios ? `, ${customization.accesorios}` : '';
    const fasciasDescription = customization.fascias ? `, ${customization.fascias}` : '';
    const modsDescription = customization.modificaciones.length > 0 
      ? `, featuring ${customization.modificaciones.join(', ')}`
      : '';

    const basePrompt = `Professional automotive photography of a ${baseDescription}, ${colorDescription}, ${finishDescription}, ${styleDescription}${detailsDescription}${accesoriosDescription}${fasciasDescription}${modsDescription}, studio lighting, ultra-detailed, 8K resolution, cinematic, photorealistic`;

    const prompts = {
      frontal: `${basePrompt}, front view, low angle shot, dramatic lighting, hero shot`,
      lateral: `${basePrompt}, side profile, dynamic perspective, motion blur background, racing environment`,
      trasero: `${basePrompt}, rear view, wide angle lens, sunset ambiance, road setting`,
      superior: `${basePrompt}, top-down view, isometric perspective, showroom floor, clean background`,
      interior: `${basePrompt}, interior cabin view, luxury materials, detailed dashboard, ambient lighting`,
      detalle: `${basePrompt}, close-up detail shot, macro lens, texture focus, artistic composition`
    };

    setGeneratedPrompts(prompts);
    addNotification('Prompts generados exitosamente', 'success');
  };

  // Generar imagen con IA
  const generateImage = async (angle) => {
    if (!generatedPrompts[angle]) {
      addNotification('Genere los prompts primero', 'error');
      return;
    }
    
    try {
      addNotification(`Generando imagen ${angle}...`, 'info');
      const imageData = await generateCarImage(generatedPrompts[angle]);
      
      setProjectImages(prev => ({
        ...prev, 
        [currentProjectId || 'default']: {
          ...prev[currentProjectId || 'default'],
          [angle]: imageData 
        }
      }));
      
      addNotification(`Imagen ${angle} generada exitosamente`, 'success');
    } catch (error) {
      addNotification('Error generando imagen: ' + error.message, 'error');
    }
  };

  // Función para agregar miembro al equipo
  const addTeamMember = (memberId) => {
    const member = teamMembers.find(m => m.id === memberId);
    if (member && !projectData.equipo.includes(memberId)) {
      setProjectData(prev => ({
        ...prev,
        equipo: [...prev.equipo, memberId]
      }));
    }
  };

  // Función para eliminar miembro del equipo
  const removeTeamMember = (memberId) => {
    setProjectData(prev => ({
      ...prev,
      equipo: prev.equipo.filter(id => id !== memberId)
    }));
  };

  // Función para exportar datos
  const exportData = () => {
    const data = {
      projects: savedProjects,
      inventory: inventory,
      reminders: reminders,
      images: projectImages,
      exportDate: new Date().toISOString()
    };
    
    const blob = new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `autodesign-backup-${new Date().toISOString().split('T')[0]}.json`;
    a.click();
    URL.revokeObjectURL(url);
    addNotification('Datos exportados exitosamente', 'success');
  };

  // Función para generar reporte PDF (simulado)
  const generatePDF = () => {
    addNotification('Generando reporte PDF...', 'info');
    // En una implementación real, usaríamos una librería como jsPDF
    setTimeout(() => {
      addNotification('Reporte PDF generado exitosamente', 'success');
    }, 2000);
  };

  // Función para compartir proyecto
  const shareProject = () => {
    navigator.clipboard.writeText(window.location.href);
    addNotification('Enlace copiado al portapapeles', 'success');
  };

  // Función para guardar proyecto
  const saveProject = () => {
    if (!vehicleData.marca || !vehicleData.modelo || !projectData.cliente) {
      addNotification('Complete los datos básicos del vehículo y cliente', 'error');
      return;
    }

    const projectId = currentProjectId || Date.now();
    const newProject = {
      id: projectId,
      date: new Date().toLocaleDateString('es-ES', { 
        year: 'numeric', 
        month: 'long', 
        day: 'numeric' 
      }),
      vehicleData,
      customization,
      projectData,
      prompts: generatedPrompts,
      costBreakdown,
      images: projectImages[projectId] || []
    };

    // Actualizar o agregar proyecto
    if (currentProjectId) {
      setSavedProjects(savedProjects.map(p => 
        p.id === currentProjectId ? newProject : p
      ));
    } else {
      setSavedProjects([...savedProjects, newProject]);
    }
    
    setCurrentProjectId(projectId);
    addNotification('Proyecto guardado exitosamente', 'success');
    
    // Agregar recordatorio automático
    if (projectData.fecha_entrega) {
      addReminder('Entrega', projectData.fecha_entrega, `Entregar ${vehicleData.marca} ${vehicleData.modelo}`);
    }
    
    // Guardar versión
    saveVersion();
  };

  // Cargar proyecto existente
  const loadProject = (project) => {
    setVehicleData({...project.vehicleData});
    setCustomization({...project.customization});
    setProjectData({...project.projectData});
    setGeneratedPrompts({...project.prompts});
    setCostBreakdown({...project.costBreakdown});
    setCurrentProjectId(project.id);
    addNotification(`Proyecto "${project.vehicleData.marca} ${project.vehicleData.modelo}" cargado`, 'success');
  };

  // Función para agregar notificación
  const addNotification = (message, type) => {
    const id = Date.now();
    setNotifications(prev => [...prev, { id, message, type }]);
    
    // Remover notificación después de 5 segundos
    setTimeout(() => {
      setNotifications(prev => prev.filter(n => n.id !== id));
    }, 5000);
  };

  // Función para agregar recordatorio
  const addReminder = (title, date, description) => {
    const newReminder = {
      id: Date.now(),
      title,
      date,
      description,
      completed: false
    };
    
    setReminders([...reminders, newReminder]);
    addNotification(`Recordatorio "${title}" agregado`, 'success');
  };

  // Función para filtrar proyectos
  const filteredProjects = savedProjects.filter(project => {
    const matchesSearch = `${project.vehicleData.marca} ${project.vehicleData.modelo} ${project.projectData.cliente}`
      .toLowerCase()
      .includes(searchTerm.toLowerCase());
    
    const matchesFilter = filterStatus === 'all' || project.projectData.estado === filterStatus;
    
    return matchesSearch && matchesFilter;
  });

  // Función para estadísticas
  const getStats = () => {
    const totalProjects = savedProjects.length;
    const completedProjects = savedProjects.filter(p => p.projectData.estado === 'Finalizado').length;
    const totalRevenue = savedProjects.reduce((sum, p) => sum + (parseFloat(p.projectData.precio_estimado) || 0), 0);
    const avgProjectValue = totalProjects > 0 ? totalRevenue / totalProjects : 0;
    
    const mostPopularColor = savedProjects.reduce((acc, project) => {
      const color = project.customization.color_principal;
      acc[color] = (acc[color] || 0) + 1;
      return acc;
    }, {});
    
    const topColor = Object.keys(mostPopularColor).reduce((a, b) => 
      mostPopularColor[a] > mostPopularColor[b] ? a : b, '#000000'
    );

    return {
      totalProjects,
      completedProjects,
      totalRevenue,
      avgProjectValue,
      topColor,
      completionRate: totalProjects > 0 ? (completedProjects / totalProjects * 100) : 0
    };
  };

  const stats = getStats();

  // Configuración de pestañas
  const tabs = [
    { id: 'vehiculo', name: 'Vehículo', icon: Car },
    { id: 'personalizar', name: 'Personalización', icon: Palette },
    { id: 'costos', name: 'Costos', icon: DollarSign },
    { id: 'prompts', name: 'Prompts IA', icon: FileText },
    { id: 'proyecto', name: 'Proyecto', icon: Calculator },
    { id: 'guardados', name: 'Proyectos', icon: Save },
    { id: 'equipo', name: 'Equipo', icon: Users }
  ];

  const themeClass = darkMode ? 'dark bg-gray-900 text-white' : 'bg-gradient-to-br from-gray-50 to-blue-50';

  return (
    <div className={`min-h-screen ${themeClass} p-4 transition-all duration-300`}>
      <div className="max-w-7xl mx-auto">
        {/* Notificaciones */}
        <NotificationCenter notifications={notifications} />
        
        {/* Header mejorado */}
        <div className={`${darkMode ? 'bg-gray-800' : 'bg-gradient-to-r from-blue-600 to-indigo-700'} rounded-xl shadow-xl p-6 mb-6 text-white`}>
          <div className="flex flex-col lg:flex-row justify-between
