<template>
    <v-dialog v-model="dialog" width="400" transition="dialog-bottom-transition" persistent>
        <v-card>
            <v-card-title class="white--text font-weight-bold fondo">
                Eliminar Usuario
                <v-spacer></v-spacer>
                <!-- boton de salir -->
                <v-scroll-x-transition>
                    <slot name="salir" v-if="showMessage && !loading"></slot>
                </v-scroll-x-transition>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text>
                <!-- mensajes -->
                <v-card elevation="0" height="50" class="mt-5">
                    <v-scroll-x-transition>
                        <v-alert border="left" colored-border elevation="2"  dense :type="type" v-show="showMessage">
                            {{mensaje}}
                        </v-alert>
                    </v-scroll-x-transition> 
                </v-card>
                
                <div class="text-center title font-weight-black py-5">
                    ¿ Seguro que quiere eliminar este Usuario ? 
                </div>

                <v-row justify="space-around" class="py-5">
                    <v-btn 
                        elevation="3" color="#232323" 
                        class="text-capitalize white--text" 
                        @click="eliminar()" :loading="loading"
                        :disabled="$parent.eliminado"
                    >
                        Sí, seguro
                    </v-btn>
                    
                    <slot name="close"></slot>
                </v-row>
            </v-card-text>
        </v-card>
    </v-dialog>
</template>

<script>
import Usuario from '@/services/Usuario';
import Clientes from '@/services/Clientes';

    export default {
        props:{
            dialog:{
                type:Boolean,
                default:false
            }
        },
        data() {
            return {
                type:'error',
                mensaje:'',
                showMessage:false,
                loading:false,
            }
        },
         watch:{
            dialog(){
                if (!this.dialog) setTimeout(() => {this.showMessage = false},400);
            }
        },
        methods:{
            respuesta(mensaje,type){
                this.mensaje = mensaje;
                this.type = type
                this.loading = false;
                this.showMessage = true;
                //setTimeout(() => {this.showMessage = false}, 2000);
            },
            eliminar(){
                if(this.$parent.bandera.perfil_id == 3) this.getCliente();
                else this.deleteUsuario();
            },
            getCliente(){
                this.loading = true;
                Clientes().get(`/?usuario_id=${this.$parent.bandera.id}`).then((response) => {
                    this.getClienteBuys(response.data.data[0].id);
                }).catch(e => {
                    console.log(e);
                    this.respuesta("Error de conección.","#D32F2F");
                });
            },
            getClienteBuys(id){
                Clientes().get(`/${id}/buys`).then((response) => {
                    if(response.data.response.data.compras == 0) this.deleteUsuario();
                    else this.respuesta("No se puede eliminar este Usuario.","#D32F2F");
                }).catch(e => {
                    console.log(e);
                    this.respuesta("Error de conección.","#D32F2F");
                });
            },
            deleteUsuario(){//elimina el grupo (solo si el grupo no tiene conceptos indexados)
                Usuario().delete(`/${this.$parent.bandera.id}`).then(() => {
                    this.$parent.eliminado = true;
                    this.respuesta("Usuario eliminado exitosamente.","success");
                    this.loading = false;
                }).catch(e => {
                    console.log(e);
                    this.respuesta("Error de conección.","#D32F2F");
                });
            },
        }
    }
</script>

<style lang="scss" scoped>
    .fondo{
        background: #1f3b63;
    }
</style>