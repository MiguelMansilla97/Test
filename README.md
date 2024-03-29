import crypto from "crypto";
import tv4 from "tv4";
import payload from "../Datos/payload.json";
import credencial from "../Datos/credenciales.json";
import moment from "moment";
//Función para mapear las credenciales

export function getHeader(cred) {
    const randomUUID = crypto.randomUUID();
  
    function mapearRecursivo(objeto) {
      const resultado = {};
  
      for (const [clave, valor] of Object.entries(objeto)) {


       if (clave==='x-idrequerimiento' && valor === 'asd') {
         
         resultado[clave]=randomUUID;
        } else if(clave==='X-Timestamp' && valor === 'random'){
          resultado[clave]=moment().format("YYYY-MM-DDTHH:mm:ss.SSS[Z]");
          
        }else{
          resultado[clave] = valor;
        }

      }
   
      return resultado;
    }
  
    const resultado = mapearRecursivo(cred);
  
    return resultado;
  }
  
  
  
  //Función reutilizable para validar todos los schemas
  export function validadorSchema(response:any,schema:any){
   
    const validator = tv4.validateResult(response,schema);
    if(validator.valid ==true){
      console.log("Validación de esquema: Exitoso")
    }else{
      console.log("Validación de esquema fallido debido a: "+ validator.error.message)
    }
    return validator.valid;
  }
  // Define la función para agregar un retraso personalizado
 export function retraso(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  //nueva consulta
  export function validarConsulta(consulta: any,headerTS :any) {
    let validator = false;
  
    for (const result of consulta) {
      //console.log("Resultado:");
  
      // Verificar si result es un array
      if (Array.isArray(result) && result.length >= 7) {
        const jsonString = result[0];
        const parsedObject = JSON.parse(jsonString);
  
        // Acceder a todas las propiedades del objeto JSON
        const cuilValue = parsedObject.cuil;
        const tokenValue = parsedObject.token;
        //const additionalInfo = result[1].split('|');
  
        // Buscar las propiedades específicas en la cadena adicional
     
        let timestampValue=result[4].split('[X-Timestamp:"');
        let timestamp= timestampValue[1].split('"]');
  
        if (cuilValue == payload.Ok.cuil && tokenValue == payload.Ok.token  &&  timestamp[0] ==headerTS) {
           
          validator = true;
        }
        
      } else {
        
        console.error("El objeto result no es un array o no tiene la cantidad esperada de elementos.");
        
      }
    }
  
    return validator;
  }
  //Función para reutilizar los casos de prueba
  export function requerido_Invalido(payload:any,error:any,parametro:any,longitud:number){
    function generarCadenaNumerica(longitud: number): string {
      return Math.floor(Math.pow(10, longitud - 1) + Math.random() * 9 * Math.pow(10, longitud - 1)).toString();
    }
    function generarCadenaAlfanumerica(longitud: number): string {
      const caracteres = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';
      let cadena = '';
      for (let i = 0; i < longitud; i++) {
        const caracterAleatorio = caracteres.charAt(Math.floor(Math.random() * caracteres.length));
        cadena += caracterAleatorio;
      }
      return cadena;
    }
    function mapearRecursivo(objeto) {
      const resultado = {};
      for (const [clave, valor] of Object.entries(objeto)) {
      switch(error){
      case "REQUERIDO":
        if(clave==parametro){
          resultado[clave] = "";
        }else{
          resultado[clave]=valor;
        }
        break;
        case "INVALIDO":
          if(clave==parametro){
            resultado[clave] = "%$%&#";
          }else{
            resultado[clave]=valor;
          }
          break;
          case"OMISION":
          if(clave !== parametro){
            resultado[clave]=valor;
          }
          break;
          case "LONGNUM":
            if(clave==parametro){
              resultado[clave] = generarCadenaNumerica(longitud);
            }else{
              resultado[clave]=valor;
            }
            break;
            case  "LONGALFNUM":
              if(clave==parametro){
                resultado[clave] = generarCadenaAlfanumerica(longitud);
              }else{
                resultado[clave]=valor;
              }
    }
  }
  return resultado;
  }
   
  const resultado = mapearRecursivo(payload);
   
  return resultado;
  }
