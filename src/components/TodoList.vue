<template>
    <div class="task-list">
        <div class="dashboard">
            <!-- Estadísticas generales -->
            <div class="stats">
                <div>
                    <h2>{{ totalProjects }}</h2>
                    <p>Proyectos Totales</p>
                </div>
                <div>
                    <h2>{{ totalTasks }}</h2>
                    <p>Tareas Totales</p>
                </div>
                <div>
                    <h2>{{ completedTasks }}</h2>
                    <p>Tareas Completadas</p>
                </div>
            </div>
        </div>



        <!-- Campo de búsqueda y filtro -->
        <div class="search-filter">
            <input class="input" type="text" v-model="searchQuery" placeholder="Buscar por nombre..." />
            <select class="input" v-model="statusFilter">
                <option value="">Todos los estados</option>
                <option value="pendiente">Pendiente</option>
                <option value="completado">Completado</option>
                <option value="en progreso">En Progreso</option>
            </select>
        </div>

        <form class="form" @submit.prevent="createProject">
            <label class="label" for="project">Nuevo proyecto:</label>
            <input class="input" type="text" v-model="newProjectName" id="project" />
            <label class="label" for="description">Descripción:</label>
            <textarea class="input" v-model="newProjectDescription" id="description"></textarea>
            <label class="label" for="status">Estado:</label>
            <select class="input" v-model="newProjectStatus" id="status">
                <option value="pendiente">Pendiente</option>
                <option value="completado">Completado</option>
                <option value="en progreso">En Progreso</option>
            </select>
            <button class="button-create" type="submit">
                <i class="fas fa-plus"></i> Crear proyecto
            </button>
        </form>

        <div class="project-card" v-for="(project, index) in filteredProjects" :key="'project' + index">
            <div class="card-header" :style="getHeaderColor(project)">
                <input :disabled="!project.editing" type="text" v-model="project.projectName"
                    class="project-name-input" />
                <button @click="toggleEdit(project)" class="edit-button">
                    <i :class="['fas', project.editing ? 'fa-save' : 'fa-edit']"></i>
                </button>
                <button @click="deleteProject(index)" class="delete-button">
                    <i class="fas fa-trash-alt"></i>
                </button>

            </div>
            <div class="card-body">
                <p class="project-description">{{ project.description }}</p>
                <p class="project-status">Estado: {{ project.status }}</p>
                <form @submit.prevent="createTask(project)">
                    <label class="label" :for="'task-' + index">Nueva tarea:</label>
                    <input class="input" type="text" v-model="newTasks[index]" :id="'task-' + index" />
                    <button class="button-task"><i class="fas fa-plus"></i> Crear tarea</button>
                </form>
                <ul class="list">
                    <li v-for="(task, i) in project.tasks" :key="'task' + i" class="task"
                        @click="completeTask(project, i)">
                        <input type="checkbox" v-model="task.completed" class="task-checkbox" />
                        <input :disabled="!task.editing" type="text" v-model="task.text" class="task-input" />
                        <button @click="toggleTaskEdit(task)" class="edit-task-button">
                            <i :class="['fas', task.editing ? 'fa-save' : 'fa-edit']"></i>
                        </button>
                        <button @click="deleteTask(project, i)" class="delete-task-button">
                            <i class="fas fa-trash-alt"></i>
                        </button>
                    </li>
                </ul>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            newProjectName: "",
            newProjectDescription: "",
            newProjectStatus: "pendiente", // Estado por defecto
            newTasks: [],
            projects: [],
            searchQuery: "",
            statusFilter: ""
        };
    },
    computed: {
        filteredProjects() {
            let filtered = this.projects.filter(project => {
                // Filtrar por nombre
                let nameMatch = project.projectName.toLowerCase().includes(this.searchQuery.toLowerCase());

                // Filtrar por estado
                let statusMatch = true;
                if (this.statusFilter) {
                    statusMatch = project.status === this.statusFilter;
                }

                return nameMatch && statusMatch;
            });

            return filtered;
        },

        totalProjects() {
            return this.projects.length;
        },
        totalTasks() {
            return this.projects.reduce((total, project) => total + project.tasks.length, 0);
        },
        completedTasks() {
            return this.projects.reduce((total, project) => {
                return total + project.tasks.filter(task => task.completed).length;
            }, 0);
        }
    },
    methods: {
        countCompletedTasks(project) {
            return project.tasks.filter(task => task.completed).length;
        },
        createProject() {
            const trimmedName = this.newProjectName.trim();

            if (trimmedName === "") {
                alert("El nombre del proyecto no puede estar vacío.");
                return;
            }

            if (this.projects.some(project => project.projectName === trimmedName)) {
                alert("¡Este proyecto ya existe!");
                return;
            }

            let project = {
                projectName: trimmedName,
                description: this.newProjectDescription,
                tasks: [],
                editing: false,
                status: this.newProjectStatus,
                active: true // Nuevo campo para estado activo/inactivo
            };

            this.projects.push(project);
            this.newProjectName = "";
            this.newProjectDescription = "";
            this.newTasks.push("");
        },
        createTask(project) {
            const index = this.projects.indexOf(project);
            if (index !== -1 && this.newTasks[index].trim() !== "") {
                if (project.tasks.some(task => task.text === this.newTasks[index].trim())) {
                    alert("¡Esta tarea ya existe en este proyecto!");
                    return;
                }

                let task = {
                    text: this.newTasks[index].trim(),
                    completed: false,
                    editing: false
                };

                project.tasks.push(task);
                this.newTasks[index] = "";
            }
        },
        completeTask(project, taskIndex) {
            const task = project.tasks[taskIndex];
            if (task) {
                task.completed = !task.completed;
            }
        },
        toggleEdit(project) {
            project.editing = !project.editing;
        },
        deleteProject(index) {
            if (confirm("¿Estás seguro de que quieres eliminar este proyecto?")) {
                this.projects.splice(index, 1);
                this.newTasks.splice(index, 1);
            }
        },
        toggleTaskEdit(task) {
            task.editing = !task.editing;
        },
        deleteTask(project, taskIndex) {
            if (confirm("¿Estás seguro de que quieres eliminar esta tarea?")) {
                project.tasks.splice(taskIndex, 1);
            }
        },
        getHeaderColor(project) {
            switch (project.status) {
                case 'completado':
                    return { backgroundColor: '#2ecc71' }; // Verde
                case 'en progreso':
                    return { backgroundColor: '#f39c12' }; // Naranja
                case 'pendiente':
                default:
                    return { backgroundColor: '#3498db' }; // Azul por defecto
            }
        },
        toggleProjectStatus(project) {
            project.active = !project.active; // Alternar estado activo/inactivo
        }
    }
};
</script>

<style scoped>
.task-list {
    width: 100%;
    max-width: 800px;
    margin: 0 auto;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
}

.form {
    background: white;
    border-radius: 12px;
    padding: 30px;
    box-shadow: 0px 10px 22px -1px rgba(0, 0, 0, 0.25);
    margin-top: 20px;
    width: 100%;
    box-sizing: border-box;
}

.label {
    display: block;
    margin-bottom: 10px;
}

.input {
    height: 35px;
    width: calc(100% - 20px);
    box-sizing: border-box;
}

.button-create,
.button-task {
    margin-top: 10px;
    height: 35px;
    width: 150px;
    border: none;
    border-radius: 5px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
    background-color: #3498db;
    color: #ecf0f1;
    cursor: pointer;
    display: flex;
    justify-content: center;
    align-items: center;
}

.project-card {
    width: calc(50% - 20px);
    margin: 10px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    overflow: hidden;
    display: flex;
    flex-direction: column;
}

.card-header {
    padding: 15px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.2);
    color: white;
}

.card-body {
    padding: 20px;
}

.list {
    margin-top: 20px;
    padding-left: 20px;
}

.task {
    cursor: pointer;
    margin-bottom: 8px;
    display: flex;
    align-items: center;
}

.task-checkbox {
    margin-right: 10px;
}

.task-input {
    flex: 1;
    border: none;
    border-bottom: 1px solid #ccc;
    padding: 5px;
    font-size: 16px;
}

.edit-button,
.delete-button,
.edit-task-button,
.delete-task-button {
    background-color: transparent;
    color: #3498db;
    border: none;
    cursor: pointer;
    font-size: 16px;
    margin-left: 5px;
}

.fa-save {
    color: #2ecc71;
}

.fa-edit {
    color: #3498db;
}

.fa-trash-alt {
    color: #e74c3c;
}

.completed {
    text-decoration: line-through;
    color: lightgrey;
}

.project-name-input {
    background-color: transparent;
    border: none;
    color: inherit;
    font-size: 20px;
    font-weight: bold;
    width: 100%;
}

.project-status {
    margin-top: 10px;
    font-weight: bold;
}


.search-filter {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
}

.dashboard {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.stats {
    display: flex;
    justify-content: space-around;
    width: 100%;
    margin-bottom: 20px;
}

.stats div {
    text-align: center;
}

.project-list {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    grid-gap: 20px;
}

.project {
    padding: 20px;
    background-color: #f9f9f9;
    border-radius: 8px;
    box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
}
</style>
