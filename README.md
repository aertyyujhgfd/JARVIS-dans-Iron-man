import speech_recognition as sr
import pyttsx3 as ttx
import pywhatkit
import datetime
import webbrowser
import pyttsx3
from googleapiclient.discovery import build
import googleapiclient.errors
import requests
from bs4 import BeautifulSoup
import os
from selenium import webdriver
import pyautogui
import time

#Importer les bibliothéques pour localiser télephone
import phonenumbers
from phonenumbers import geocoder
from phonenumbers import carrier
from opencage.geocoder import OpenCageGeocode
import requests

listener=sr.Recognizer()
engine=ttx.init()
voice=engine.getProperty('voices')
engine.setProperty('voice','french')

#Initialisation pour ecouter ce que dit l'utilisateur
def parler(text):
    engine.say(text)
    engine.runAndWait()

#Commande pour ecouter et interpreter la personne
def ecouter():
    try:
        with sr.Microphone() as source:
            voix=listener.listen(source)
            print("parlez maintenant")
            command=listener.recognize_google(voix,language='fr-FR')
            command.lower()
            
    except:
        pass
        import command
    return command

def lancer_assistant():
    command= ecouter()
    
#Commande pour ouvrir une music sur Youtube
    if 'chanson' in command:
        chanteur=command.replace('mets la chanson de','')
        print(chanteur)
        pywhatkit.playonyt(chanteur)
        
#Commande pour donner l'heure
    elif 'heure' in command:
        import command
        import datetime
        heure=datetime.datetime.now().strftime('%H:%M')
        parler('il est'+heure)


#Commande de reponse à Bonjour 
    elif 'bonjour' in command:
        import command
        parler('bonjour,comment aller vous Thibault?')

#Commande pour rechercher sur internet des information 


#Commande pour l'opérateur du télephone et ou il ce situe  
    elif 'téléphone' in command:
        import command
    #Trouver le pays du numéro
        print("écriver votre numéro de téléphone avec +33 devant")
        num=input("")
        monNum= phonenumbers.parse(num)
        localisation= geocoder.description_for_number(monNum,"fr")
        print(localisation)

        #Trouver l'opérateur mobile
        operateur= phonenumbers.parse(num)
        print(carrier.name_for_number(operateur,"fr"))
    
    elif 'b' in command:
        import command
    #Trouver le pays du numéro
        from openai.api_resources.abstract.api_resource import APIResource
        from openai.api_resources.abstract.createable_api_resource import CreateableAPIResource
        from openai.api_resources.abstract.deletable_api_resource import DeletableAPIResource
        from openai.api_resources.abstract.listable_api_resource import ListableAPIResource
        from openai.api_resources.abstract.nested_resource_class_methods import (
            nested_resource_class_methods,
        )
        from openai.api_resources.abstract.updateable_api_resource import UpdateableAPIResource

        #api_resource.py
        ######################################################################################
#Ce code définit la classe "APIResource" qui étend la classe "OpenAIObject". La classe "APIResource" fournit des
#méthodes pour interagir avec des ressources sur le serveur OpenAI via une API REST.
#La méthode "retrieve" permet de récupérer une instance d'une ressource en fonction de son ID,
#tandis que la méthode "refresh" permet de mettre à jour les informations d'une instance existante à partir des données sur le
#serveur. La méthode "class_url" renvoie l'URL pour la classe en question.
#Des variables de classe telles que "api_prefix" et "azure_api_prefix" sont définies pour des fins de préfixage de l'URL de l'API.
    ######################################################################################

        from urllib.parse import quote_plus

        import openai
        from openai import api_requestor, error, util
        from openai.openai_object import OpenAIObject
        from openai.util import ApiType
        from typing import Optional


        class APIResource(OpenAIObject):
            api_prefix = ""
            azure_api_prefix = "openai"
            azure_deployments_prefix = "deployments"

            @classmethod
            def retrieve(
                cls, id, api_key=None, request_id=None, request_timeout=None, **params
            ):
                instance = cls(id, api_key, **params)
                instance.refresh(request_id=request_id, request_timeout=request_timeout)
                return instance

            @classmethod
            def aretrieve(
                cls, id, api_key=None, request_id=None, request_timeout=None, **params
            ):
                instance = cls(id, api_key, **params)
                return instance.arefresh(request_id=request_id, request_timeout=request_timeout)

            def refresh(self, request_id=None, request_timeout=None):
                self.refresh_from(
                    self.request(
                        "get",
                        self.instance_url(),
                        request_id=request_id,
                        request_timeout=request_timeout,
                    )
                )
                return self

            async def arefresh(self, request_id=None, request_timeout=None):
                self.refresh_from(
                    await self.arequest(
                        "get",
                        self.instance_url(operation="refresh"),
                        request_id=request_id,
                        request_timeout=request_timeout,
                    )
                )
                return self

            @classmethod
            def class_url(cls):
                if cls == APIResource:
                    raise NotImplementedError(
                        "APIResource is an abstract class. You should perform actions on its subclasses."
                    )
                # Les espaces de noms sont séparés dans les noms d'objets par des points (.) et dans les URL.
                # avec des barres obliques (/), donc remplacez les premières par les secondes.
                base = cls.OBJECT_NAME.replace(".", "/")  # type: ignore
                if cls.api_prefix:
                    return "/%s/%s" % (cls.api_prefix, base)
                return "/%s" % (base)

            def instance_url(self, operation=None):
                id = self.get("id")

                if not isinstance(id, str):
                    raise error.InvalidRequestError(
                        "Could not determine which URL to request: %s instance "
                        "has invalid ID: %r, %s. ID should be of type `str` (or"
                        " `unicode`)" % (type(self).__name__, id, type(id)),
                        "id",
                    )
                api_version = self.api_version or openai.api_version
                extn = quote_plus(id)

                if self.typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    if not api_version:
                        raise error.InvalidRequestError(
                            "An API version is required for the Azure API type."
                        )

                    if not operation:
                        base = self.class_url()
                        return "/%s%s/%s?api-version=%s" % (
                            self.azure_api_prefix,
                            base,
                            extn,
                            api_version,
                        )

                    return "/%s/%s/%s/%s?api-version=%s" % (
                        self.azure_api_prefix,
                        self.azure_deployments_prefix,
                        extn,
                        operation,
                        api_version,
                    )

                elif self.typed_api_type == ApiType.OPEN_AI:
                    base = self.class_url()
                    return "%s/%s" % (base, extn)

                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % self.api_type)

            # Les arguments `méthode_` et `url_` sont suffixés par un trait de soulignement pour
            # éviter les conflits avec les paramètres réels de la requête dans `params`.
            @classmethod
            def _static_request(
                cls,
                method_,
                url_,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_version=api_version,
                    organization=organization,
                    api_base=api_base,
                    api_type=api_type,
                )
                response, _, api_key = requestor.request(
                    method_, url_, params, request_id=request_id
                )
                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )

            @classmethod
            async def _astatic_request(
                cls,
                method_,
                url_,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_version=api_version,
                    organization=organization,
                    api_base=api_base,
                    api_type=api_type,
                )
                response, _, api_key = await requestor.arequest(
                    method_, url_, params, request_id=request_id
                )
                return response

            @classmethod
            def _get_api_type_and_version(
                cls, api_type: Optional[str] = None, api_version: Optional[str] = None
            ):
                typed_api_type = (
                    ApiType.from_str(api_type)
                    if api_type
                    else ApiType.from_str(openai.api_type)
                )
                typed_api_version = api_version or openai.api_version
                return (typed_api_type, typed_api_version)


        ##############################################################################"
        ###################################################################

        from openai import api_requestor, util, error
        from openai.api_resources.abstract.api_resource import APIResource
        from openai.util import ApiType


        class CreateableAPIResource(APIResource):
            plain_old_data = False

            @classmethod
            def __prepare_create_requestor(
                cls,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )
                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )

                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    base = cls.class_url()
                    url = "/%s%s?api-version=%s" % (cls.azure_api_prefix, base, api_version)
                elif typed_api_type == ApiType.OPEN_AI:
                    url = cls.class_url()
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)
                return requestor, url

            @classmethod
            def create(
                cls,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url = cls.__prepare_create_requestor(
                    api_key,
                    api_base,
                    api_type,
                    api_version,
                    organization,
                )

                response, _, api_key = requestor.request(
                    "post", url, params, request_id=request_id
                )

                return util.convert_to_openai_object(
                    response,
                    api_key,
                    api_version,
                    organization,
                    plain_old_data=cls.plain_old_data,
                )

            @classmethod
            async def acreate(
                cls,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url = cls.__prepare_create_requestor(
                    api_key,
                    api_base,
                    api_type,
                    api_version,
                    organization,
                )

                response, _, api_key = await requestor.arequest(
                    "post", url, params, request_id=request_id
                )

                return util.convert_to_openai_object(
                    response,
                    api_key,
                    api_version,
                    organization,
                    plain_old_data=cls.plain_old_data,
                )

        ############################################################################
            "deletable_api_resource.py"
            ################################################################

        from urllib.parse import quote_plus
        from typing import Awaitable

        from openai import error
        from openai.api_resources.abstract.api_resource import APIResource
        from openai.util import ApiType


        class DeletableAPIResource(APIResource):
            @classmethod
            def __prepare_delete(cls, sid, api_type=None, api_version=None):
                if isinstance(cls, APIResource):
                    raise ValueError(".delete ne peut être appelé qu'en tant que méthode de classe maintenant.")

                base = cls.class_url()
                extn = quote_plus(sid)

                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )
                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    url = "/%s%s/%s?api-version=%s" % (
                        cls.azure_api_prefix,
                        base,
                        extn,
                        api_version,
                    )
                elif typed_api_type == ApiType.OPEN_AI:
                    url = "%s/%s" % (base, extn)
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)
                return url

            @classmethod
            def delete(cls, sid, api_type=None, api_version=None, **params):
                url = cls.__prepare_delete(sid, api_type, api_version)

                return cls._static_request(
                    "delete", url, api_type=api_type, api_version=api_version, **params
                )

            @classmethod
            def adelete(cls, sid, api_type=None, api_version=None, **params) -> Awaitable:
                url = cls.__prepare_delete(sid, api_type, api_version)

                return cls._astatic_request(
                    "delete", url, api_type=api_type, api_version=api_version, **params
                )



        ##############################################################################
        "engine_api_resource.py"
        ##########################################################################

        import time
        from pydoc import apropos
        from typing import Optional
        from urllib.parse import quote_plus

        import openai
        from openai import api_requestor, error, util
        from openai.api_resources.abstract.api_resource import APIResource
        from openai.openai_response import OpenAIResponse
        from openai.util import ApiType

        MAX_TIMEOUT = 20


        class EngineAPIResource(APIResource):
            plain_old_data = False

            def __init__(self, engine: Optional[str] = None, **kwargs):
                super().__init__(engine=engine, **kwargs)

            @classmethod
            def class_url(
                cls,
                engine: Optional[str] = None,
                api_type: Optional[str] = None,
                api_version: Optional[str] = None,
            ):
                # Les espaces de noms sont séparés dans les noms d'objets par des points (.) et dans les URL.
                # avec des barres obliques (/), donc remplacez les premières par les secondes.

                base = cls.OBJECT_NAME.replace(".", "/")  # type: ignore
                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )

                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    if not api_version:
                        raise error.InvalidRequestError(
                            "An API version is required for the Azure API type."
                        )
                    if engine is None:
                        raise error.InvalidRequestError(
                            "You must provide the deployment name in the 'engine' parameter to access the Azure OpenAI service"
                        )
                    extn = quote_plus(engine)
                    return "/%s/%s/%s/%s?api-version=%s" % (
                        cls.azure_api_prefix,
                        cls.azure_deployments_prefix,
                        extn,
                        base,
                        api_version,
                    )

                elif typed_api_type == ApiType.OPEN_AI:
                    if engine is None:
                        return "/%s" % (base)

                    extn = quote_plus(engine)
                    return "/engines/%s/%s" % (extn, base)

                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)

            @classmethod
            def __prepare_create_request(
                cls,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):
                deployment_id = params.pop("deployment_id", None)
                engine = params.pop("engine", deployment_id)
                model = params.get("model", None)
                timeout = params.pop("timeout", None)
                stream = params.get("stream", False)
                headers = params.pop("headers", None)
                request_timeout = params.pop("request_timeout", None)
                typed_api_type = cls._get_api_type_and_version(api_type=api_type)[0]
                if typed_api_type in (util.ApiType.AZURE, util.ApiType.AZURE_AD):
                    if deployment_id is None and engine is None:
                        raise error.InvalidRequestError(
                            "Must provide an 'engine' or 'deployment_id' parameter to create a %s"
                            % cls,
                            "engine",
                        )
                else:
                    if model is None and engine is None:
                        raise error.InvalidRequestError(
                            "Must provide an 'engine' or 'model' parameter to create a %s"
                            % cls,
                            "engine",
                        )

                if timeout is None:
                    # No special timeout handling
                    pass
                elif timeout > 0:
                    # API only supports timeouts up to MAX_TIMEOUT
                    params["timeout"] = min(timeout, MAX_TIMEOUT)
                    timeout = (timeout - params["timeout"]) or None
                elif timeout == 0:
                    params["timeout"] = MAX_TIMEOUT

                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )
                url = cls.class_url(engine, api_type, api_version)
                return (
                    deployment_id,
                    engine,
                    timeout,
                    stream,
                    headers,
                    request_timeout,
                    typed_api_type,
                    requestor,
                    url,
                    params,
                )

            @classmethod
            def create(
                cls,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                (
                    deployment_id,
                    engine,
                    timeout,
                    stream,
                    headers,
                    request_timeout,
                    typed_api_type,
                    requestor,
                    url,
                    params,
                ) = cls.__prepare_create_request(
                    api_key, api_base, api_type, api_version, organization, **params
                )

                response, _, api_key = requestor.request(
                    "post",
                    url,
                    params=params,
                    headers=headers,
                    stream=stream,
                    request_id=request_id,
                    request_timeout=request_timeout,
                )

                if stream:
                    # must be an iterator
                    assert not isinstance(response, OpenAIResponse)
                    return (
                        util.convert_to_openai_object(
                            line,
                            api_key,
                            api_version,
                            organization,
                            engine=engine,
                            plain_old_data=cls.plain_old_data,
                        )
                        for line in response
                    )
                else:
                    obj = util.convert_to_openai_object(
                        response,
                        api_key,
                        api_version,
                        organization,
                        engine=engine,
                        plain_old_data=cls.plain_old_data,
                    )

                    if timeout is not None:
                        obj.wait(timeout=timeout or None)

                return obj

            @classmethod
            async def acreate(
                cls,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                (
                    deployment_id,
                    engine,
                    timeout,
                    stream,
                    headers,
                    request_timeout,
                    typed_api_type,
                    requestor,
                    url,
                    params,
                ) = cls.__prepare_create_request(
                    api_key, api_base, api_type, api_version, organization, **params
                )
                response, _, api_key = await requestor.arequest(
                    "post",
                    url,
                    params=params,
                    headers=headers,
                    stream=stream,
                    request_id=request_id,
                    request_timeout=request_timeout,
                )

                if stream:
                    # must be an iterator
                    assert not isinstance(response, OpenAIResponse)
                    return (
                        util.convert_to_openai_object(
                            line,
                            api_key,
                            api_version,
                            organization,
                            engine=engine,
                            plain_old_data=cls.plain_old_data,
                        )
                        async for line in response
                    )
                else:
                    obj = util.convert_to_openai_object(
                        response,
                        api_key,
                        api_version,
                        organization,
                        engine=engine,
                        plain_old_data=cls.plain_old_data,
                    )

                    if timeout is not None:
                        await obj.await_(timeout=timeout or None)

                return obj

            def instance_url(self):
                id = self.get("id")

                if not isinstance(id, str):
                    raise error.InvalidRequestError(
                        f"Could not determine which URL to request: {type(self).__name__} instance has invalid ID: {id}, {type(id)}. ID should be of type str.",
                        "id",
                    )

                extn = quote_plus(id)
                params_connector = "?"

                if self.typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    api_version = self.api_version or openai.api_version
                    if not api_version:
                        raise error.InvalidRequestError(
                            "An API version is required for the Azure API type."
                        )
                    base = self.OBJECT_NAME.replace(".", "/")
                    url = "/%s/%s/%s/%s/%s?api-version=%s" % (
                        self.azure_api_prefix,
                        self.azure_deployments_prefix,
                        self.engine,
                        base,
                        extn,
                        api_version,
                    )
                    params_connector = "&"

                elif self.typed_api_type == ApiType.OPEN_AI:
                    base = self.class_url(self.engine, self.api_type, self.api_version)
                    url = "%s/%s" % (base, extn)

                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % self.api_type)

                timeout = self.get("timeout")
                if timeout is not None:
                    timeout = quote_plus(str(timeout))
                    url += params_connector + "timeout={}".format(timeout)
                return url

            def wait(self, timeout=None):
                start = time.time()
                while self.status != "complete":
                    self.timeout = (
                        min(timeout + start - time.time(), MAX_TIMEOUT)
                        if timeout is not None
                        else MAX_TIMEOUT
                    )
                    if self.timeout < 0:
                        del self.timeout
                        break
                    self.refresh()
                return self

            async def await_(self, timeout=None):
                """Async version of `EngineApiResource.wait`"""
                start = time.time()
                while self.status != "complete":
                    self.timeout = (
                        min(timeout + start - time.time(), MAX_TIMEOUT)
                        if timeout is not None
                        else MAX_TIMEOUT
                    )
                    if self.timeout < 0:
                        del self.timeout
                        break
                    await self.arefresh()
                return self


        #############################################################################
        "listable_api_resource.py"
        ##############################################################################

        from openai import api_requestor, util, error
        from openai.api_resources.abstract.api_resource import APIResource
        from openai.util import ApiType


        class ListableAPIResource(APIResource):
            @classmethod
            def auto_paging_iter(cls, *args, **params):
                return cls.list(*args, **params).auto_paging_iter()

            @classmethod
            def __prepare_list_requestor(
                cls,
                api_key=None,
                api_version=None,
                organization=None,
                api_base=None,
                api_type=None,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base or cls.api_base(),
                    api_version=api_version,
                    api_type=api_type,
                    organization=organization,
                )

                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )

                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    base = cls.class_url()
                    url = "/%s%s?api-version=%s" % (cls.azure_api_prefix, base, api_version)
                elif typed_api_type == ApiType.OPEN_AI:
                    url = cls.class_url()
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)
                return requestor, url

            @classmethod
            def list(
                cls,
                api_key=None,
                request_id=None,
                api_version=None,
                organization=None,
                api_base=None,
                api_type=None,
                **params,
            ):
                requestor, url = cls.__prepare_list_requestor(
                    api_key,
                    api_version,
                    organization,
                    api_base,
                    api_type,
                )

                response, _, api_key = requestor.request(
                    "get", url, params, request_id=request_id
                )
                openai_object = util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )
                openai_object._retrieve_params = params
                return openai_object

            @classmethod
            async def alist(
                cls,
                api_key=None,
                request_id=None,
                api_version=None,
                organization=None,
                api_base=None,
                api_type=None,
                **params,
            ):
                requestor, url = cls.__prepare_list_requestor(
                    api_key,
                    api_version,
                    organization,
                    api_base,
                    api_type,
                )

                response, _, api_key = await requestor.arequest(
                    "get", url, params, request_id=request_id
                )
                openai_object = util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )
                openai_object._retrieve_params = params
                return openai_object

        #########################################################################################
        "nested_resource_class_methods.py"
        #########################################################################################

        from urllib.parse import quote_plus

        from openai import api_requestor, util


        def _nested_resource_class_methods(
            resource,
            path=None,
            operations=None,
            resource_plural=None,
            async_=False,
        ):
            if resource_plural is None:
                resource_plural = "%ss" % resource
            if path is None:
                path = resource_plural
            if operations is None:
                raise ValueError("operations list required")

            def wrapper(cls):
                def nested_resource_url(cls, id, nested_id=None):
                    url = "%s/%s/%s" % (cls.class_url(), quote_plus(id), quote_plus(path))
                    if nested_id is not None:
                        url += "/%s" % quote_plus(nested_id)
                    return url

                resource_url_method = "%ss_url" % resource
                setattr(cls, resource_url_method, classmethod(nested_resource_url))

                def nested_resource_request(
                    cls,
                    method,
                    url,
                    api_key=None,
                    request_id=None,
                    api_version=None,
                    organization=None,
                    **params,
                ):
                    requestor = api_requestor.APIRequestor(
                        api_key, api_version=api_version, organization=organization
                    )
                    response, _, api_key = requestor.request(
                        method, url, params, request_id=request_id
                    )
                    return util.convert_to_openai_object(
                        response, api_key, api_version, organization
                    )

                async def anested_resource_request(
                    cls,
                    method,
                    url,
                    api_key=None,
                    request_id=None,
                    api_version=None,
                    organization=None,
                    **params,
                ):
                    requestor = api_requestor.APIRequestor(
                        api_key, api_version=api_version, organization=organization
                    )
                    response, _, api_key = await requestor.arequest(
                        method, url, params, request_id=request_id
                    )
                    return util.convert_to_openai_object(
                        response, api_key, api_version, organization
                    )

                resource_request_method = "%ss_request" % resource
                setattr(
                    cls,
                    resource_request_method,
                    classmethod(
                        anested_resource_request if async_ else nested_resource_request
                    ),
                )

                for operation in operations:
                    if operation == "create":

                        def create_nested_resource(cls, id, **params):
                            url = getattr(cls, resource_url_method)(id)
                            return getattr(cls, resource_request_method)("post", url, **params)

                        create_method = "create_%s" % resource
                        setattr(cls, create_method, classmethod(create_nested_resource))

                    elif operation == "retrieve":

                        def retrieve_nested_resource(cls, id, nested_id, **params):
                            url = getattr(cls, resource_url_method)(id, nested_id)
                            return getattr(cls, resource_request_method)("get", url, **params)

                        retrieve_method = "retrieve_%s" % resource
                        setattr(cls, retrieve_method, classmethod(retrieve_nested_resource))

                    elif operation == "update":

                        def modify_nested_resource(cls, id, nested_id, **params):
                            url = getattr(cls, resource_url_method)(id, nested_id)
                            return getattr(cls, resource_request_method)("post", url, **params)

                        modify_method = "modify_%s" % resource
                        setattr(cls, modify_method, classmethod(modify_nested_resource))

                    elif operation == "delete":

                        def delete_nested_resource(cls, id, nested_id, **params):
                            url = getattr(cls, resource_url_method)(id, nested_id)
                            return getattr(cls, resource_request_method)(
                                "delete", url, **params
                            )

                        delete_method = "delete_%s" % resource
                        setattr(cls, delete_method, classmethod(delete_nested_resource))

                    elif operation == "list":

                        def list_nested_resources(cls, id, **params):
                            url = getattr(cls, resource_url_method)(id)
                            return getattr(cls, resource_request_method)("get", url, **params)

                        list_method = "list_%s" % resource_plural
                        setattr(cls, list_method, classmethod(list_nested_resources))

                    else:
                        raise ValueError("Unknown operation: %s" % operation)

                return cls

            return wrapper


        def nested_resource_class_methods(
            resource,
            path=None,
            operations=None,
            resource_plural=None,
        ):
            return _nested_resource_class_methods(
                resource, path, operations, resource_plural, async_=False
            )


        def anested_resource_class_methods(
            resource,
            path=None,
            operations=None,
            resource_plural=None,
        ):
            return _nested_resource_class_methods(
                resource, path, operations, resource_plural, async_=True
            )

        ###########################################################################################
        "updateable_api_resource.py"
        ##########################################################################################

        from urllib.parse import quote_plus
        from typing import Awaitable

        from openai.api_resources.abstract.api_resource import APIResource


        class UpdateableAPIResource(APIResource):
            @classmethod
            def modify(cls, sid, **params):
                url = "%s/%s" % (cls.class_url(), quote_plus(sid))
                return cls._static_request("post", url, **params)

            @classmethod
            def amodify(cls, sid, **params) -> Awaitable:
                url = "%s/%s" % (cls.class_url(), quote_plus(sid))
                return cls._astatic_request("patch", url, **params)


        ############################################################################
        "__init__.py"
            ################################################################

        from openai.api_resources.experimental.completion_config import (  # noqa: F401
            CompletionConfig,
        )

        from openai.api_resources.completion import Completion  # noqa: F401
        from openai.api_resources.customer import Customer  # noqa: F401
        from openai.api_resources.deployment import Deployment  # noqa: F401
        from openai.api_resources.edit import Edit  # noqa: F401
        from openai.api_resources.embedding import Embedding  # noqa: F401
        from openai.api_resources.engine import Engine  # noqa: F401
        from openai.api_resources.error_object import ErrorObject  # noqa: F401
        from openai.api_resources.file import File  # noqa: F401
        from openai.api_resources.fine_tune import FineTune  # noqa: F401
        from openai.api_resources.image import Image  # noqa: F401
        from openai.api_resources.model import Model  # noqa: F401
        from openai.api_resources.moderation import Moderation  # noqa: F401

        ############################################################################
        "completion_config.py"
            ################################################################

        from openai.api_resources.abstract import (
            CreateableAPIResource,
            DeletableAPIResource,
            ListableAPIResource,
        )


        class CompletionConfig(
            CreateableAPIResource, ListableAPIResource, DeletableAPIResource
        ):
            OBJECT_NAME = "experimental.completion_configs"


        import time

        from openai import util
        from openai.api_resources.abstract import DeletableAPIResource, ListableAPIResource
        from openai.api_resources.abstract.engine_api_resource import EngineAPIResource
        from openai.error import TryAgain


        class Completion(EngineAPIResource):
            OBJECT_NAME = "completions"

            @classmethod
            def create(cls, *args, **kwargs):
                """
                Creates a new completion for the provided prompt and parameters.

                See https://beta.openai.com/docs/api-reference/completions/create for a list
                of valid parameters.
                """
                start = time.time()
                timeout = kwargs.pop("timeout", None)

                while True:
                    try:
                        return super().create(*args, **kwargs)
                    except TryAgain as e:
                        if timeout is not None and time.time() > start + timeout:
                            raise

                        util.log_info("Waiting for model to warm up", error=e)

            @classmethod
            async def acreate(cls, *args, **kwargs):
                """
                Creates a new completion for the provided prompt and parameters.

                See https://beta.openai.com/docs/api-reference/completions/create for a list
                of valid parameters.
                """
                start = time.time()
                timeout = kwargs.pop("timeout", None)

                while True:
                    try:
                        return await super().acreate(*args, **kwargs)
                    except TryAgain as e:
                        if timeout is not None and time.time() > start + timeout:
                            raise

                        util.log_info("Waiting for model to warm up", error=e)

        ############################################################################
        "customer.py "
            ################################################################
        from openai.openai_object import OpenAIObject


        class Customer(OpenAIObject):
            @classmethod
            def get_url(self, customer, endpoint):
                return f"/customer/{customer}/{endpoint}"

            @classmethod
            def create(cls, customer, endpoint, **params):
                instance = cls()
                return instance.request("post", cls.get_url(customer, endpoint), params)

            @classmethod
            def acreate(cls, customer, endpoint, **params):
                instance = cls()
                return instance.arequest("post", cls.get_url(customer, endpoint), params)

        ############################################################################
        "customer.py " "objet"
            ################################################################
        from openai.openai_object import OpenAIObject


        class Customer(OpenAIObject):
            @classmethod
            def get_url(self, customer, endpoint):
                return f"/customer/{customer}/{endpoint}"

            @classmethod
            def create(cls, customer, endpoint, **params):
                instance = cls()
                return instance.request("post", cls.get_url(customer, endpoint), params)

            @classmethod
            def acreate(cls, customer, endpoint, **params):
                instance = cls()
                return instance.arequest("post", cls.get_url(customer, endpoint), params)

        ############################################################################
        "deployment.py  "
            ################################################################
        from openai import util
        from openai.api_resources.abstract import (
            DeletableAPIResource,
            ListableAPIResource,
            CreateableAPIResource,
        )
        from openai.error import InvalidRequestError, APIError


        class Deployment(CreateableAPIResource, ListableAPIResource, DeletableAPIResource):
            OBJECT_NAME = "deployments"

            @classmethod
            def _check_create(cls, *args, **kwargs):
                typed_api_type, _ = cls._get_api_type_and_version(
                    kwargs.get("api_type", None), None
                )
                if typed_api_type not in (util.ApiType.AZURE, util.ApiType.AZURE_AD):
                    raise APIError(
                        "Deployment operations are only available for the Azure API type."
                    )

                if kwargs.get("model", None) is None:
                    raise InvalidRequestError(
                        "Must provide a 'model' parameter to create a Deployment.",
                        param="model",
                    )

                scale_settings = kwargs.get("scale_settings", None)
                if scale_settings is None:
                    raise InvalidRequestError(
                        "Must provide a 'scale_settings' parameter to create a Deployment.",
                        param="scale_settings",
                    )

                if "scale_type" not in scale_settings or (
                    scale_settings["scale_type"].lower() == "manual"
                    and "capacity" not in scale_settings
                ):
                    raise InvalidRequestError(
                        "The 'scale_settings' parameter contains invalid or incomplete values.",
                        param="scale_settings",
                    )

            @classmethod
            def create(cls, *args, **kwargs):
                """
                Creates a new deployment for the provided prompt and parameters.
                """
                cls._check_create(*args, **kwargs)
                return super().create(*args, **kwargs)

            @classmethod
            def acreate(cls, *args, **kwargs):
                """
                Creates a new deployment for the provided prompt and parameters.
                """
                cls._check_create(*args, **kwargs)
                return super().acreate(*args, **kwargs)

            @classmethod
            def _check_list(cls, *args, **kwargs):
                typed_api_type, _ = cls._get_api_type_and_version(
                    kwargs.get("api_type", None), None
                )
                if typed_api_type not in (util.ApiType.AZURE, util.ApiType.AZURE_AD):
                    raise APIError(
                        "Deployment operations are only available for the Azure API type."
                    )

            @classmethod
            def list(cls, *args, **kwargs):
                cls._check_list(*args, **kwargs)
                return super().list(*args, **kwargs)

            @classmethod
            def alist(cls, *args, **kwargs):
                cls._check_list(*args, **kwargs)
                return super().alist(*args, **kwargs)

            @classmethod
            def _check_delete(cls, *args, **kwargs):
                typed_api_type, _ = cls._get_api_type_and_version(
                    kwargs.get("api_type", None), None
                )
                if typed_api_type not in (util.ApiType.AZURE, util.ApiType.AZURE_AD):
                    raise APIError(
                        "Deployment operations are only available for the Azure API type."
                    )

            @classmethod
            def delete(cls, *args, **kwargs):
                cls._check_delete(*args, **kwargs)
                return super().delete(*args, **kwargs)

            @classmethod
            def adelete(cls, *args, **kwargs):
                cls._check_delete(*args, **kwargs)
                return super().adelete(*args, **kwargs)

            @classmethod
            def _check_retrieve(cls, *args, **kwargs):
                typed_api_type, _ = cls._get_api_type_and_version(
                    kwargs.get("api_type", None), None
                )
                if typed_api_type not in (util.ApiType.AZURE, util.ApiType.AZURE_AD):
                    raise APIError(
                        "Deployment operations are only available for the Azure API type."
                    )

            @classmethod
            def retrieve(cls, *args, **kwargs):
                cls._check_retrieve(*args, **kwargs)
                return super().retrieve(*args, **kwargs)

            @classmethod
            def aretrieve(cls, *args, **kwargs):
                cls._check_retrieve(*args, **kwargs)
                return super().aretrieve(*args, **kwargs)


        ############################################################################
        " edit.py "
            ################################################################
        import time

        from openai import util, error
        from openai.api_resources.abstract.engine_api_resource import EngineAPIResource
        from openai.error import TryAgain


        class Edit(EngineAPIResource):
            OBJECT_NAME = "edits"

            @classmethod
            def create(cls, *args, **kwargs):
                """
                Creates a new edit for the provided input, instruction, and parameters.
                """
                start = time.time()
                timeout = kwargs.pop("timeout", None)

                api_type = kwargs.pop("api_type", None)
                typed_api_type = cls._get_api_type_and_version(api_type=api_type)[0]
                if typed_api_type in (util.ApiType.AZURE, util.ApiType.AZURE_AD):
                    raise error.InvalidAPIType(
                        "This operation is not supported by the Azure OpenAI API yet."
                    )

                while True:
                    try:
                        return super().create(*args, **kwargs)
                    except TryAgain as e:
                        if timeout is not None and time.time() > start + timeout:
                            raise

                        util.log_info("Waiting for model to warm up", error=e)

            @classmethod
            async def acreate(cls, *args, **kwargs):
                """
                Creates a new edit for the provided input, instruction, and parameters.
                """
                start = time.time()
                timeout = kwargs.pop("timeout", None)

                api_type = kwargs.pop("api_type", None)
                typed_api_type = cls._get_api_type_and_version(api_type=api_type)[0]
                if typed_api_type in (util.ApiType.AZURE, util.ApiType.AZURE_AD):
                    raise error.InvalidAPIType(
                        "This operation is not supported by the Azure OpenAI API yet."
                    )

                while True:
                    try:
                        return await super().acreate(*args, **kwargs)
                    except TryAgain as e:
                        if timeout is not None and time.time() > start + timeout:
                            raise

                        util.log_info("Waiting for model to warm up", error=e)

        ############################################################################
        " embedding.py "
            ################################################################

        import base64
        import time


        from openai import util
        from openai.api_resources.abstract.engine_api_resource import EngineAPIResource
        from openai.datalib import numpy as np, assert_has_numpy
        from openai.error import TryAgain


        class Embedding(EngineAPIResource):
            OBJECT_NAME = "embeddings"

            @classmethod
            def create(cls, *args, **kwargs):
                """
                Creates a new embedding for the provided input and parameters.

                See https://beta.openai.com/docs/api-reference/embeddings for a list
                of valid parameters.
                """
                start = time.time()
                timeout = kwargs.pop("timeout", None)

                user_provided_encoding_format = kwargs.get("encoding_format", None)

                # If encoding format was not explicitly specified, we opaquely use base64 for performance
                if not user_provided_encoding_format:
                    kwargs["encoding_format"] = "base64"

                while True:
                    try:
                        response = super().create(*args, **kwargs)

                        # If a user specifies base64, we'll just return the encoded string.
                        # This is only for the default case.
                        if not user_provided_encoding_format:
                            for data in response.data:

                                # If an engine isn't using this optimization, don't do anything
                                if type(data["embedding"]) == str:
                                    assert_has_numpy()
                                    data["embedding"] = np.frombuffer(
                                        base64.b64decode(data["embedding"]), dtype="float32"
                                    ).tolist()

                        return response
                    except TryAgain as e:
                        if timeout is not None and time.time() > start + timeout:
                            raise

                        util.log_info("Waiting for model to warm up", error=e)

            @classmethod
            async def acreate(cls, *args, **kwargs):
                """
                Creates a new embedding for the provided input and parameters.

                See https://beta.openai.com/docs/api-reference/embeddings for a list
                of valid parameters.
                """
                start = time.time()
                timeout = kwargs.pop("timeout", None)

                user_provided_encoding_format = kwargs.get("encoding_format", None)

                # If encoding format was not explicitly specified, we opaquely use base64 for performance
                if not user_provided_encoding_format:
                    kwargs["encoding_format"] = "base64"

                while True:
                    try:
                        response = await super().acreate(*args, **kwargs)

                        # If a user specifies base64, we'll just return the encoded string.
                        # This is only for the default case.
                        if not user_provided_encoding_format:
                            for data in response.data:

                                # If an engine isn't using this optimization, don't do anything
                                if type(data["embedding"]) == str:
                                    data["embedding"] = np.frombuffer(
                                        base64.b64decode(data["embedding"]), dtype="float32"
                                    ).tolist()

                        return response
                    except TryAgain as e:
                        if timeout is not None and time.time() > start + timeout:
                            raise

                        util.log_info("Waiting for model to warm up", error=e)

        ############################################################################
        " engine.py"
            ################################################################

        import time
        import warnings

        from openai import util
        from openai.api_resources.abstract import ListableAPIResource, UpdateableAPIResource
        from openai.error import TryAgain


        class Engine(ListableAPIResource, UpdateableAPIResource):
            OBJECT_NAME = "engines"

            def generate(self, timeout=None, **params):
                start = time.time()
                while True:
                    try:
                        return self.request(
                            "post",
                            self.instance_url() + "/generate",
                            params,
                            stream=params.get("stream"),
                            plain_old_data=True,
                        )
                    except TryAgain as e:
                        if timeout is not None and time.time() > start + timeout:
                            raise

                        util.log_info("Waiting for model to warm up", error=e)

            async def agenerate(self, timeout=None, **params):
                start = time.time()
                while True:
                    try:
                        return await self.arequest(
                            "post",
                            self.instance_url() + "/generate",
                            params,
                            stream=params.get("stream"),
                            plain_old_data=True,
                        )
                    except TryAgain as e:
                        if timeout is not None and time.time() > start + timeout:
                            raise

                        util.log_info("Waiting for model to warm up", error=e)

            def embeddings(self, **params):
                warnings.warn(
                    "Engine.embeddings is deprecated, use Embedding.create", DeprecationWarning
                )
                return self.request("post", self.instance_url() + "/embeddings", params)

        ############################################################################
        " error_object.py "
            ################################################################

        from typing import Optional

        from openai.openai_object import OpenAIObject
        from openai.util import merge_dicts


        class ErrorObject(OpenAIObject):
            def refresh_from(
                self,
                values,
                api_key=None,
                api_version=None,
                api_type=None,
                organization=None,
                response_ms: Optional[int] = None,
            ):
                # Unlike most other API resources, the API will omit attributes in
                # error objects when they have a null value. We manually set default
                # values here to facilitate generic error handling.
                values = merge_dicts({"message": None, "type": None}, values)
                return super(ErrorObject, self).refresh_from(
                    values=values,
                    api_key=api_key,
                    api_version=api_version,
                    api_type=api_type,
                    organization=organization,
                    response_ms=response_ms,
                )

        ############################################################################
        " file.py"
            ################################################################
        import json
        import os
        from typing import cast

        import openai
        from openai import api_requestor, util, error
        from openai.api_resources.abstract import DeletableAPIResource, ListableAPIResource
        from openai.util import ApiType


        class File(ListableAPIResource, DeletableAPIResource):
            OBJECT_NAME = "files"

            @classmethod
            def __prepare_file_create(
                cls,
                file,
                purpose,
                model=None,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                user_provided_filename=None,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base or openai.api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )
                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )

                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    base = cls.class_url()
                    url = "/%s%s?api-version=%s" % (cls.azure_api_prefix, base, api_version)
                elif typed_api_type == ApiType.OPEN_AI:
                    url = cls.class_url()
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)

                # Set the filename on 'purpose' and 'model' to None so they are
                # interpreted as form data.
                files = [("purpose", (None, purpose))]
                if model is not None:
                    files.append(("model", (None, model)))
                if user_provided_filename is not None:
                    files.append(
                        ("file", (user_provided_filename, file, "application/octet-stream"))
                    )
                else:
                    files.append(("file", ("file", file, "application/octet-stream")))

                return requestor, url, files

            @classmethod
            def create(
                cls,
                file,
                purpose,
                model=None,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                user_provided_filename=None,
            ):
                requestor, url, files = cls.__prepare_file_create(
                    file,
                    purpose,
                    model,
                    api_key,
                    api_base,
                    api_type,
                    api_version,
                    organization,
                    user_provided_filename,
                )
                response, _, api_key = requestor.request("post", url, files=files)
                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )

            @classmethod
            async def acreate(
                cls,
                file,
                purpose,
                model=None,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                user_provided_filename=None,
            ):
                requestor, url, files = cls.__prepare_file_create(
                    file,
                    purpose,
                    model,
                    api_key,
                    api_base,
                    api_type,
                    api_version,
                    organization,
                    user_provided_filename,
                )
                response, _, api_key = await requestor.arequest("post", url, files=files)
                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )

            @classmethod
            def __prepare_file_download(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base or openai.api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )
                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )

                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    base = cls.class_url()
                    url = "/%s%s/%s?api-version=%s" % (
                        cls.azure_api_prefix,
                        base,
                        id,
                        api_version,
                    )
                elif typed_api_type == ApiType.OPEN_AI:
                    url = "%s/%s" % (cls.class_url(), id)
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)

                return requestor, url

            @classmethod
            def download(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
            ):
                requestor, url = cls.__prepare_file_download(
                    id, api_key, api_base, api_type, api_version, organization
                )

                result = requestor.request_raw("get", url)
                if not 200 <= result.status_code < 300:
                    raise requestor.handle_error_response(
                        result.content,
                        result.status_code,
                        json.loads(cast(bytes, result.content)),
                        result.headers,
                        stream_error=False,
                    )
                return result.content

            @classmethod
            async def adownload(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
            ):
                requestor, url = cls.__prepare_file_download(
                    id, api_key, api_base, api_type, api_version, organization
                )

                async with api_requestor.aiohttp_session() as session:
                    result = await requestor.arequest_raw("get", url, session)
                    if not 200 <= result.status < 300:
                        raise requestor.handle_error_response(
                            result.content,
                            result.status,
                            json.loads(cast(bytes, result.content)),
                            result.headers,
                            stream_error=False,
                        )
                    return result.content

            @classmethod
            def __find_matching_files(cls, name, all_files, purpose):
                matching_files = []
                basename = os.path.basename(name)
                for f in all_files:
                    if f["purpose"] != purpose:
                        continue
                    file_basename = os.path.basename(f["filename"])
                    if file_basename != basename:
                        continue
                    if "bytes" in f and f["bytes"] != bytes:
                        continue
                    if "size" in f and int(f["size"]) != bytes:
                        continue
                    matching_files.append(f)
                return matching_files

            @classmethod
            def find_matching_files(
                cls,
                name,
                bytes,
                purpose,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
            ):
                """Find already uploaded files with the same name, size, and purpose."""
                all_files = cls.list(
                    api_key=api_key,
                    api_base=api_base or openai.api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                ).get("data", [])
                return cls.__find_matching_files(name, all_files, purpose)

            @classmethod
            async def afind_matching_files(
                cls,
                name,
                bytes,
                purpose,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
            ):
                """Find already uploaded files with the same name, size, and purpose."""
                all_files = (
                    await cls.alist(
                        api_key=api_key,
                        api_base=api_base or openai.api_base,
                        api_type=api_type,
                        api_version=api_version,
                        organization=organization,
                    )
                ).get("data", [])
                return cls.__find_matching_files(name, all_files, purpose)

        ############################################################################
        " fine_tune.py "
            ################################################################

        from urllib.parse import quote_plus

        from openai import api_requestor, util, error
        from openai.api_resources.abstract import (
            CreateableAPIResource,
            ListableAPIResource,
            nested_resource_class_methods,
        )
        from openai.api_resources.abstract.deletable_api_resource import DeletableAPIResource
        from openai.openai_response import OpenAIResponse
        from openai.util import ApiType


        @nested_resource_class_methods("event", operations=["list"])
        class FineTune(ListableAPIResource, CreateableAPIResource, DeletableAPIResource):
            OBJECT_NAME = "fine-tunes"

            @classmethod
            def _prepare_cancel(
                cls,
                id,
                api_key=None,
                api_type=None,
                request_id=None,
                api_version=None,
                **params,
            ):
                base = cls.class_url()
                extn = quote_plus(id)

                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )
                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    url = "/%s%s/%s/cancel?api-version=%s" % (
                        cls.azure_api_prefix,
                        base,
                        extn,
                        api_version,
                    )
                elif typed_api_type == ApiType.OPEN_AI:
                    url = "%s/%s/cancel" % (base, extn)
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)

                instance = cls(id, api_key, **params)
                return instance, url

            @classmethod
            def cancel(
                cls,
                id,
                api_key=None,
                api_type=None,
                request_id=None,
                api_version=None,
                **params,
            ):
                instance, url = cls._prepare_cancel(
                    id,
                    api_key,
                    api_type,
                    request_id,
                    api_version,
                    **params,
                )
                return instance.request("post", url, request_id=request_id)

            @classmethod
            def acancel(
                cls,
                id,
                api_key=None,
                api_type=None,
                request_id=None,
                api_version=None,
                **params,
            ):
                instance, url = cls._prepare_cancel(
                    id,
                    api_key,
                    api_type,
                    request_id,
                    api_version,
                    **params,
                )
                return instance.arequest("post", url, request_id=request_id)

            @classmethod
            def _prepare_stream_events(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                base = cls.class_url()
                extn = quote_plus(id)

                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )

                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )

                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    url = "/%s%s/%s/events?stream=true&api-version=%s" % (
                        cls.azure_api_prefix,
                        base,
                        extn,
                        api_version,
                    )
                elif typed_api_type == ApiType.OPEN_AI:
                    url = "%s/%s/events?stream=true" % (base, extn)
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)

                return requestor, url

            @classmethod
            def stream_events(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url = cls._prepare_stream_events(
                    id,
                    api_key,
                    api_base,
                    api_type,
                    request_id,
                    api_version,
                    organization,
                    **params,
                )

                response, _, api_key = requestor.request(
                    "get", url, params, stream=True, request_id=request_id
                )

                assert not isinstance(response, OpenAIResponse)  # must be an iterator
                return (
                    util.convert_to_openai_object(
                        line,
                        api_key,
                        api_version,
                        organization,
                    )
                    for line in response
                )

            @classmethod
            async def astream_events(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url = cls._prepare_stream_events(
                    id,
                    api_key,
                    api_base,
                    api_type,
                    request_id,
                    api_version,
                    organization,
                    **params,
                )

                response, _, api_key = await requestor.arequest(
                    "get", url, params, stream=True, request_id=request_id
                )

                assert not isinstance(response, OpenAIResponse)  # must be an iterator
                return (
                    util.convert_to_openai_object(
                        line,
                        api_key,
                        api_version,
                        organization,
                    )
                    async for line in response
                )

        ############################################################################
        " fine_tune.py "
            ################################################################

        from urllib.parse import quote_plus

        from openai import api_requestor, util, error
        from openai.api_resources.abstract import (
            CreateableAPIResource,
            ListableAPIResource,
            nested_resource_class_methods,
        )
        from openai.api_resources.abstract.deletable_api_resource import DeletableAPIResource
        from openai.openai_response import OpenAIResponse
        from openai.util import ApiType


        @nested_resource_class_methods("event", operations=["list"])
        class FineTune(ListableAPIResource, CreateableAPIResource, DeletableAPIResource):
            OBJECT_NAME = "fine-tunes"

            @classmethod
            def _prepare_cancel(
                cls,
                id,
                api_key=None,
                api_type=None,
                request_id=None,
                api_version=None,
                **params,
            ):
                base = cls.class_url()
                extn = quote_plus(id)

                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )
                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    url = "/%s%s/%s/cancel?api-version=%s" % (
                        cls.azure_api_prefix,
                        base,
                        extn,
                        api_version,
                    )
                elif typed_api_type == ApiType.OPEN_AI:
                    url = "%s/%s/cancel" % (base, extn)
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)

                instance = cls(id, api_key, **params)
                return instance, url

            @classmethod
            def cancel(
                cls,
                id,
                api_key=None,
                api_type=None,
                request_id=None,
                api_version=None,
                **params,
            ):
                instance, url = cls._prepare_cancel(
                    id,
                    api_key,
                    api_type,
                    request_id,
                    api_version,
                    **params,
                )
                return instance.request("post", url, request_id=request_id)

            @classmethod
            def acancel(
                cls,
                id,
                api_key=None,
                api_type=None,
                request_id=None,
                api_version=None,
                **params,
            ):
                instance, url = cls._prepare_cancel(
                    id,
                    api_key,
                    api_type,
                    request_id,
                    api_version,
                    **params,
                )
                return instance.arequest("post", url, request_id=request_id)

            @classmethod
            def _prepare_stream_events(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                base = cls.class_url()
                extn = quote_plus(id)

                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )

                typed_api_type, api_version = cls._get_api_type_and_version(
                    api_type, api_version
                )

                if typed_api_type in (ApiType.AZURE, ApiType.AZURE_AD):
                    url = "/%s%s/%s/events?stream=true&api-version=%s" % (
                        cls.azure_api_prefix,
                        base,
                        extn,
                        api_version,
                    )
                elif typed_api_type == ApiType.OPEN_AI:
                    url = "%s/%s/events?stream=true" % (base, extn)
                else:
                    raise error.InvalidAPIType("Unsupported API type %s" % api_type)

                return requestor, url

            @classmethod
            def stream_events(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url = cls._prepare_stream_events(
                    id,
                    api_key,
                    api_base,
                    api_type,
                    request_id,
                    api_version,
                    organization,
                    **params,
                )

                response, _, api_key = requestor.request(
                    "get", url, params, stream=True, request_id=request_id
                )

                assert not isinstance(response, OpenAIResponse)  # must be an iterator
                return (
                    util.convert_to_openai_object(
                        line,
                        api_key,
                        api_version,
                        organization,
                    )
                    for line in response
                )

            @classmethod
            async def astream_events(
                cls,
                id,
                api_key=None,
                api_base=None,
                api_type=None,
                request_id=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url = cls._prepare_stream_events(
                    id,
                    api_key,
                    api_base,
                    api_type,
                    request_id,
                    api_version,
                    organization,
                    **params,
                )

                response, _, api_key = await requestor.arequest(
                    "get", url, params, stream=True, request_id=request_id
                )

                assert not isinstance(response, OpenAIResponse)  # must be an iterator
                return (
                    util.convert_to_openai_object(
                        line,
                        api_key,
                        api_version,
                        organization,
                    )
                    async for line in response
                )

        ############################################################################
        " image.py "
            ################################################################
        # WARNING: This interface is considered experimental and may changed in the future without warning.
        from typing import Any, List

        import openai
        from openai import api_requestor, util
        from openai.api_resources.abstract import APIResource


        class Image(APIResource):
            OBJECT_NAME = "images"

            @classmethod
            def _get_url(cls, action):
                return cls.class_url() + f"/{action}"

            @classmethod
            def create(
                cls,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base or openai.api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )

                _, api_version = cls._get_api_type_and_version(api_type, api_version)

                response, _, api_key = requestor.request(
                    "post", cls._get_url("generations"), params
                )

                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )

            @classmethod
            async def acreate(
                cls,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):

                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base or openai.api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )

                _, api_version = cls._get_api_type_and_version(api_type, api_version)

                response, _, api_key = await requestor.arequest(
                    "post", cls._get_url("generations"), params
                )

                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )

            @classmethod
            def _prepare_create_variation(
                cls,
                image,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base or openai.api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )
                _, api_version = cls._get_api_type_and_version(api_type, api_version)

                url = cls._get_url("variations")

                files: List[Any] = []
                for key, value in params.items():
                    files.append((key, (None, value)))
                files.append(("image", ("image", image, "application/octet-stream")))
                return requestor, url, files

            @classmethod
            def create_variation(
                cls,
                image,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url, files = cls._prepare_create_variation(
                    image,
                    api_key,
                    api_base,
                    api_type,
                    api_version,
                    organization,
                    **params,
                )

                response, _, api_key = requestor.request("post", url, files=files)

                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )

            @classmethod
            async def acreate_variation(
                cls,
                image,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url, files = cls._prepare_create_variation(
                    image,
                    api_key,
                    api_base,
                    api_type,
                    api_version,
                    organization,
                    **params,
                )

                response, _, api_key = await requestor.arequest("post", url, files=files)

                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )

            @classmethod
            def _prepare_create_edit(
                cls,
                image,
                mask=None,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor = api_requestor.APIRequestor(
                    api_key,
                    api_base=api_base or openai.api_base,
                    api_type=api_type,
                    api_version=api_version,
                    organization=organization,
                )
                _, api_version = cls._get_api_type_and_version(api_type, api_version)

                url = cls._get_url("edits")

                files: List[Any] = []
                for key, value in params.items():
                    files.append((key, (None, value)))
                files.append(("image", ("image", image, "application/octet-stream")))
                if mask is not None:
                    files.append(("mask", ("mask", mask, "application/octet-stream")))
                return requestor, url, files

            @classmethod
            def create_edit(
                cls,
                image,
                mask=None,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url, files = cls._prepare_create_edit(
                    image,
                    mask,
                    api_key,
                    api_base,
                    api_type,
                    api_version,
                    organization,
                    **params,
                )

                response, _, api_key = requestor.request("post", url, files=files)

                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )

            @classmethod
            async def acreate_edit(
                cls,
                image,
                mask=None,
                api_key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
                **params,
            ):
                requestor, url, files = cls._prepare_create_edit(
                    image,
                    mask,
                    api_key,
                    api_base,
                    api_type,
                    api_version,
                    organization,
                    **params,
                )

                response, _, api_key = await requestor.arequest("post", url, files=files)

                return util.convert_to_openai_object(
                    response, api_key, api_version, organization
                )


        ############################################################################
        " model.py "
            ################################################################

        from openai.api_resources.abstract import DeletableAPIResource, ListableAPIResource


        class Model(ListableAPIResource, DeletableAPIResource):
            OBJECT_NAME = "models"

        ############################################################################
        " moderation.py "
            ################################################################
        from typing import List, Optional, Union

        from openai.openai_object import OpenAIObject


        class Moderation(OpenAIObject):
            VALID_MODEL_NAMES: List[str] = ["text-moderation-stable", "text-moderation-latest"]

            @classmethod
            def get_url(self):
                return "/moderations"

            @classmethod
            def _prepare_create(cls, input, model, api_key):
                if model is not None and model not in cls.VALID_MODEL_NAMES:
                    raise ValueError(
                        f"The parameter model should be chosen from {cls.VALID_MODEL_NAMES} "
                        f"and it is default to be None."
                    )

                instance = cls(api_key=api_key)
                params = {"input": input}
                if model is not None:
                    params["model"] = model
                return instance, params

            @classmethod
            def create(
                cls,
                input: Union[str, List[str]],
                model: Optional[str] = None,
                api_key: Optional[str] = None,
            ):
                instance, params = cls._prepare_create(input, model, api_key)
                return instance.request("post", cls.get_url(), params)

            @classmethod
            def acreate(
                cls,
                input: Union[str, List[str]],
                model: Optional[str] = None,
                api_key: Optional[str] = None,
            ):
                instance, params = cls._prepare_create(input, model, api_key)
                return instance.arequest("post", cls.get_url(), params)
            
        ############################################################################
        " test_endpoints.py  "
            ################################################################
        import io
        import json

        import pytest
        from aiohttp import ClientSession

        import openai
        from openai import error


        pytestmark = [pytest.mark.asyncio]


        # FILE TESTS
        async def test_file_upload():
            result = await openai.File.acreate(
                file=io.StringIO(json.dumps({"text": "test file data"})),
                purpose="search",
            )
            assert result.purpose == "search"
            assert "id" in result

            result = await openai.File.aretrieve(id=result.id)
            assert result.status == "uploaded"


        # COMPLETION TESTS
        async def test_completions():
            result = await openai.Completion.acreate(
                prompt="This was a test", n=5, engine="ada"
            )
            assert len(result.choices) == 5


        async def test_completions_multiple_prompts():
            result = await openai.Completion.acreate(
                prompt=["This was a test", "This was another test"], n=5, engine="ada"
            )
            assert len(result.choices) == 10


        async def test_completions_model():
            result = await openai.Completion.acreate(prompt="This was a test", n=5, model="ada")
            assert len(result.choices) == 5
            assert result.model.startswith("ada")


        async def test_timeout_raises_error():
            # A query that should take awhile to return
            with pytest.raises(error.Timeout):
                await openai.Completion.acreate(
                    prompt="test" * 1000,
                    n=10,
                    model="ada",
                    max_tokens=100,
                    request_timeout=0.01,
                )


        async def test_timeout_does_not_error():
            # A query that should be fast
            await openai.Completion.acreate(
                prompt="test",
                model="ada",
                request_timeout=10,
            )


        async def test_completions_stream_finishes_global_session():
            async with ClientSession() as session:
                openai.aiosession.set(session)

                # A query that should be fast
                parts = []
                async for part in await openai.Completion.acreate(
                    prompt="test", model="ada", request_timeout=3, stream=True
                ):
                    parts.append(part)
                assert len(parts) > 1


        async def test_completions_stream_finishes_local_session():
            # A query that should be fast
            parts = []
            async for part in await openai.Completion.acreate(
                prompt="test", model="ada", request_timeout=3, stream=True
            ):
                parts.append(part)
            assert len(parts) > 1

        ############################################################################
        " test_api_requestor.py  "
            ################################################################

        import json

        import pytest
        import requests
        from pytest_mock import MockerFixture

        from openai import Model
        from openai.api_requestor import APIRequestor


        @pytest.mark.requestor
        def test_requestor_sets_request_id(mocker: MockerFixture) -> None:
            # Fake out 'requests' and confirm that the X-Request-Id header is set.

            got_headers = {}

            def fake_request(self, *args, **kwargs):
                nonlocal got_headers
                got_headers = kwargs["headers"]
                r = requests.Response()
                r.status_code = 200
                r.headers["content-type"] = "application/json"
                r._content = json.dumps({}).encode("utf-8")
                return r

            mocker.patch("requests.sessions.Session.request", fake_request)
            fake_request_id = "1234"
            Model.retrieve("xxx", request_id=fake_request_id)  # arbitrary API resource
            got_request_id = got_headers.get("X-Request-Id")
            assert got_request_id == fake_request_id


        @pytest.mark.requestor
        def test_requestor_open_ai_headers() -> None:
            api_requestor = APIRequestor(key="test_key", api_type="open_ai")
            headers = {"Test_Header": "Unit_Test_Header"}
            headers = api_requestor.request_headers(
                method="get", extra=headers, request_id="test_id"
            )
            assert "Test_Header" in headers
            assert headers["Test_Header"] == "Unit_Test_Header"
            assert "Authorization" in headers
            assert headers["Authorization"] == "Bearer test_key"


        @pytest.mark.requestor
        def test_requestor_azure_headers() -> None:
            api_requestor = APIRequestor(key="test_key", api_type="azure")
            headers = {"Test_Header": "Unit_Test_Header"}
            headers = api_requestor.request_headers(
                method="get", extra=headers, request_id="test_id"
            )
            assert "Test_Header" in headers
            assert headers["Test_Header"] == "Unit_Test_Header"
            assert "api-key" in headers
            assert headers["api-key"] == "test_key"


        @pytest.mark.requestor
        def test_requestor_azure_ad_headers() -> None:
            api_requestor = APIRequestor(key="test_key", api_type="azure_ad")
            headers = {"Test_Header": "Unit_Test_Header"}
            headers = api_requestor.request_headers(
                method="get", extra=headers, request_id="test_id"
            )
            assert "Test_Header" in headers
            assert headers["Test_Header"] == "Unit_Test_Header"
            assert "Authorization" in headers
            assert headers["Authorization"] == "Bearer test_key"

        ############################################################################
        " test_endpoints.py "
            ################################################################

        import io
        import json

        import pytest

        import openai
        from openai import error


        # FILE TESTS
        def test_file_upload():
            result = openai.File.create(
                file=io.StringIO(
                    json.dumps({"prompt": "test file data", "completion": "tada"})
                ),
                purpose="fine-tune",
            )
            assert result.purpose == "fine-tune"
            assert "id" in result

            result = openai.File.retrieve(id=result.id)
            assert result.status == "uploaded"


        # COMPLETION TESTS
        def test_completions():
            result = openai.Completion.create(prompt="This was a test", n=5, engine="ada")
            assert len(result.choices) == 5


        def test_completions_multiple_prompts():
            result = openai.Completion.create(
                prompt=["This was a test", "This was another test"], n=5, engine="ada"
            )
            assert len(result.choices) == 10


        def test_completions_model():
            result = openai.Completion.create(prompt="This was a test", n=5, model="ada")
            assert len(result.choices) == 5
            assert result.model.startswith("ada")


        def test_timeout_raises_error():
            # A query that should take awhile to return
            with pytest.raises(error.Timeout):
                openai.Completion.create(
                    prompt="test" * 1000,
                    n=10,
                    model="ada",
                    max_tokens=100,
                    request_timeout=0.01,
                )


        def test_timeout_does_not_error():
            # A query that should be fast
            openai.Completion.create(
                prompt="test",
                model="ada",
                request_timeout=10,
            )

        ############################################################################
        " test_file_cli.py "
            ################################################################

        import json
        import subprocess
        import time
        from tempfile import NamedTemporaryFile

        STILL_PROCESSING = "File is still processing. Check back later."


        def test_file_cli() -> None:
            contents = json.dumps({"prompt": "1 + 3 =", "completion": "4"}) + "\n"
            with NamedTemporaryFile(suffix=".jsonl", mode="wb") as train_file:
                train_file.write(contents.encode("utf-8"))
                train_file.flush()
                create_output = subprocess.check_output(
                    ["openai", "api", "files.create", "-f", train_file.name, "-p", "fine-tune"]
                )
            file_obj = json.loads(create_output)
            assert file_obj["bytes"] == len(contents)
            file_id: str = file_obj["id"]
            assert file_id.startswith("file-")
            start_time = time.time()
            while True:
                delete_result = subprocess.run(
                    ["openai", "api", "files.delete", "-i", file_id],
                    stdout=subprocess.PIPE,
                    stderr=subprocess.PIPE,
                    encoding="utf-8",
                )
                if delete_result.returncode == 0:
                    break
                elif STILL_PROCESSING in delete_result.stderr:
                    time.sleep(0.5)
                    if start_time + 60 < time.time():
                        raise RuntimeError("timed out waiting for file to become available")
                    continue
                else:
                    raise RuntimeError(
                        f"delete failed: stdout={delete_result.stdout} stderr={delete_result.stderr}"
                    )
        ############################################################################
        " test_long_examples_validator.py  "
            ################################################################

        import json
        import subprocess
        from tempfile import NamedTemporaryFile

        import pytest

        from openai.datalib import HAS_PANDAS, HAS_NUMPY, NUMPY_INSTRUCTIONS, PANDAS_INSTRUCTIONS


        @pytest.mark.skipif(not HAS_PANDAS, reason=PANDAS_INSTRUCTIONS)
        @pytest.mark.skipif(not HAS_NUMPY, reason=NUMPY_INSTRUCTIONS)
        def test_long_examples_validator() -> None:
            """
            Ensures that long_examples_validator() handles previously applied recommendations,
            namely dropped duplicates, without resulting in a KeyError.
            """

            # data
            short_prompt = "a prompt "
            long_prompt = short_prompt * 500

            short_completion = "a completion "
            long_completion = short_completion * 500

            # the order of these matters
            unprepared_training_data = [
                {"prompt": long_prompt, "completion": long_completion},  # 1 of 2 duplicates
                {"prompt": short_prompt, "completion": short_completion},
                {"prompt": long_prompt, "completion": long_completion},  # 2 of 2 duplicates
            ]

            with NamedTemporaryFile(suffix="jsonl", mode="w") as training_data:
                for prompt_completion_row in unprepared_training_data:
                    training_data.write(json.dumps(prompt_completion_row) + "\n")
                    training_data.flush()

                prepared_data_cmd_output = subprocess.run(
                    [f"openai tools fine_tunes.prepare_data -f {training_data.name}"],
                    stdout=subprocess.PIPE,
                    text=True,
                    input="y\ny\ny\ny\ny",  # apply all recommendations, one at a time
                    stderr=subprocess.PIPE,
                    encoding="utf-8",
                    shell=True,
                )

            # validate data was prepared successfully
            assert prepared_data_cmd_output.stderr == ""
            # validate get_long_indexes() applied during optional_fn() call in long_examples_validator()
            assert "indices of the long examples has changed" in prepared_data_cmd_output.stdout
            
            return prepared_data_cmd_output.stdout

        ############################################################################
        " test_url_composition.py  "
            ################################################################
        from sys import api_version

        import pytest

        from openai import Completion, Engine
        from openai.util import ApiType


        @pytest.mark.url
        def test_completions_url_composition_azure() -> None:
            url = Completion.class_url("test_engine", "azure", "2021-11-01-preview")
            assert (
                url
                == "/openai/deployments/test_engine/completions?api-version=2021-11-01-preview"
            )


        @pytest.mark.url
        def test_completions_url_composition_azure_ad() -> None:
            url = Completion.class_url("test_engine", "azure_ad", "2021-11-01-preview")
            assert (
                url
                == "/openai/deployments/test_engine/completions?api-version=2021-11-01-preview"
            )


        @pytest.mark.url
        def test_completions_url_composition_default() -> None:
            url = Completion.class_url("test_engine")
            assert url == "/engines/test_engine/completions"


        @pytest.mark.url
        def test_completions_url_composition_open_ai() -> None:
            url = Completion.class_url("test_engine", "open_ai")
            assert url == "/engines/test_engine/completions"


        @pytest.mark.url
        def test_completions_url_composition_invalid_type() -> None:
            with pytest.raises(Exception):
                url = Completion.class_url("test_engine", "invalid")


        @pytest.mark.url
        def test_completions_url_composition_instance_url_azure() -> None:
            completion = Completion(
                id="test_id",
                engine="test_engine",
                api_type="azure",
                api_version="2021-11-01-preview",
            )
            url = completion.instance_url()
            assert (
                url
                == "/openai/deployments/test_engine/completions/test_id?api-version=2021-11-01-preview"
            )


        @pytest.mark.url
        def test_completions_url_composition_instance_url_azure_ad() -> None:
            completion = Completion(
                id="test_id",
                engine="test_engine",
                api_type="azure_ad",
                api_version="2021-11-01-preview",
            )
            url = completion.instance_url()
            assert (
                url
                == "/openai/deployments/test_engine/completions/test_id?api-version=2021-11-01-preview"
            )


        @pytest.mark.url
        def test_completions_url_composition_instance_url_azure_no_version() -> None:
            completion = Completion(
                id="test_id", engine="test_engine", api_type="azure", api_version=None
            )
            with pytest.raises(Exception):
                completion.instance_url()


        @pytest.mark.url
        def test_completions_url_composition_instance_url_default() -> None:
            completion = Completion(id="test_id", engine="test_engine")
            url = completion.instance_url()
            assert url == "/engines/test_engine/completions/test_id"


        @pytest.mark.url
        def test_completions_url_composition_instance_url_open_ai() -> None:
            completion = Completion(
                id="test_id",
                engine="test_engine",
                api_type="open_ai",
                api_version="2021-11-01-preview",
            )
            url = completion.instance_url()
            assert url == "/engines/test_engine/completions/test_id"


        @pytest.mark.url
        def test_completions_url_composition_instance_url_invalid() -> None:
            completion = Completion(id="test_id", engine="test_engine", api_type="invalid")
            with pytest.raises(Exception):
                url = completion.instance_url()


        @pytest.mark.url
        def test_completions_url_composition_instance_url_timeout_azure() -> None:
            completion = Completion(
                id="test_id",
                engine="test_engine",
                api_type="azure",
                api_version="2021-11-01-preview",
            )
            completion["timeout"] = 12
            url = completion.instance_url()
            assert (
                url
                == "/openai/deployments/test_engine/completions/test_id?api-version=2021-11-01-preview&timeout=12"
            )


        @pytest.mark.url
        def test_completions_url_composition_instance_url_timeout_openai() -> None:
            completion = Completion(id="test_id", engine="test_engine", api_type="open_ai")
            completion["timeout"] = 12
            url = completion.instance_url()
            assert url == "/engines/test_engine/completions/test_id?timeout=12"


        @pytest.mark.url
        def test_engine_search_url_composition_azure() -> None:
            engine = Engine(id="test_id", api_type="azure", api_version="2021-11-01-preview")
            assert engine.api_type == "azure"
            assert engine.typed_api_type == ApiType.AZURE
            url = engine.instance_url("test_operation")
            assert (
                url
                == "/openai/deployments/test_id/test_operation?api-version=2021-11-01-preview"
            )


        @pytest.mark.url
        def test_engine_search_url_composition_azure_ad() -> None:
            engine = Engine(id="test_id", api_type="azure_ad", api_version="2021-11-01-preview")
            assert engine.api_type == "azure_ad"
            assert engine.typed_api_type == ApiType.AZURE_AD
            url = engine.instance_url("test_operation")
            assert (
                url
                == "/openai/deployments/test_id/test_operation?api-version=2021-11-01-preview"
            )


        @pytest.mark.url
        def test_engine_search_url_composition_azure_no_version() -> None:
            engine = Engine(id="test_id", api_type="azure", api_version=None)
            assert engine.api_type == "azure"
            assert engine.typed_api_type == ApiType.AZURE
            with pytest.raises(Exception):
                engine.instance_url("test_operation")


        @pytest.mark.url
        def test_engine_search_url_composition_azure_no_operation() -> None:
            engine = Engine(id="test_id", api_type="azure", api_version="2021-11-01-preview")
            assert engine.api_type == "azure"
            assert engine.typed_api_type == ApiType.AZURE
            assert (
                engine.instance_url()
                == "/openai/engines/test_id?api-version=2021-11-01-preview"
            )


        @pytest.mark.url
        def test_engine_search_url_composition_default() -> None:
            engine = Engine(id="test_id")
            assert engine.api_type == None
            assert engine.typed_api_type == ApiType.OPEN_AI
            url = engine.instance_url()
            assert url == "/engines/test_id"


        @pytest.mark.url
        def test_engine_search_url_composition_open_ai() -> None:
            engine = Engine(id="test_id", api_type="open_ai")
            assert engine.api_type == "open_ai"
            assert engine.typed_api_type == ApiType.OPEN_AI
            url = engine.instance_url()
            assert url == "/engines/test_id"


        @pytest.mark.url
        def test_engine_search_url_composition_invalid_type() -> None:
            engine = Engine(id="test_id", api_type="invalid")
            assert engine.api_type == "invalid"
            with pytest.raises(Exception):
                assert engine.typed_api_type == ApiType.OPEN_AI


        @pytest.mark.url
        def test_engine_search_url_composition_invalid_search() -> None:
            engine = Engine(id="test_id", api_type="invalid")
            assert engine.api_type == "invalid"
            with pytest.raises(Exception):
                engine.search()

        ############################################################################
        " test_util.py  "
            ################################################################

        from tempfile import NamedTemporaryFile

        import pytest

        import openai
        from openai import util


        @pytest.fixture(scope="function")
        def api_key_file():
            saved_path = openai.api_key_path
            try:
                with NamedTemporaryFile(prefix="openai-api-key", mode="wt") as tmp:
                    openai.api_key_path = tmp.name
                    yield tmp
            finally:
                openai.api_key_path = saved_path


        def test_openai_api_key_path(api_key_file) -> None:
            print("sk-foo", file=api_key_file)
            api_key_file.flush()
            assert util.default_api_key() == "sk-foo"


        def test_openai_api_key_path_with_malformed_key(api_key_file) -> None:
            print("malformed-api-key", file=api_key_file)
            api_key_file.flush()
            with pytest.raises(ValueError, match="Malformed API key"):
                util.default_api_key()

        ############################################################################
        " __init__.py  "
            ################################################################

        # OpenAI Python bindings.
        #
        # Originally forked from the MIT-licensed Stripe Python bindings.

        import os
        from contextvars import ContextVar
        from typing import Optional, TYPE_CHECKING

        from openai.api_resources import (
            Completion,
            Customer,
            Edit,
            Deployment,
            Embedding,
            Engine,
            ErrorObject,
            File,
            FineTune,
            Image,
            Model,
            Moderation,
        )
        from openai.error import APIError, InvalidRequestError, OpenAIError

        if TYPE_CHECKING:
            from aiohttp import ClientSession

        api_key = os.environ.get("OPENAI_API_KEY")
        # Path of a file with an API key, whose contents can change. Supercedes
        # `api_key` if set.  The main use case is volume-mounted Kubernetes secrets,
        # which are updated automatically.
        api_key_path: Optional[str] = os.environ.get("OPENAI_API_KEY_PATH")

        organization = os.environ.get("OPENAI_ORGANIZATION")
        api_base = os.environ.get("OPENAI_API_BASE", "https://api.openai.com/v1")
        api_type = os.environ.get("OPENAI_API_TYPE", "open_ai")
        api_version = (
            "2022-12-01" if api_type in ("azure", "azure_ad", "azuread") else None
        )
        verify_ssl_certs = True  # No effect. Certificates are always verified.
        proxy = None
        app_info = None
        enable_telemetry = False  # Ignored; the telemetry feature was removed.
        ca_bundle_path = None  # No longer used, feature was removed
        debug = False
        log = None  # Set to either 'debug' or 'info', controls console logging

        aiosession: ContextVar[Optional["ClientSession"]] = ContextVar(
            "aiohttp-session", default=None
        )  # Acts as a global aiohttp ClientSession that reuses connections.
        # This is user-supplied; otherwise, a session is remade for each request.

        __all__ = [
            "APIError",
            "Completion",
            "Customer",
            "Edit",
            "Image",
            "Deployment",
            "Embedding",
            "Engine",
            "ErrorObject",
            "File",
            "FineTune",
            "InvalidRequestError",
            "Model",
            "Moderation",
            "OpenAIError",
            "api_base",
            "api_key",
            "api_type",
            "api_key_path",
            "api_version",
            "app_info",
            "ca_bundle_path",
            "debug",
            "enable_elemetry",
            "log",
            "organization",
            "proxy",
            "verify_ssl_certs",
        ]

        ############################################################################
        "_openai_scripts.py "
            ################################################################

        #!/usr/bin/env python
        import argparse
        import logging
        import sys

        import openai
        from openai.cli import api_register, display_error, tools_register, wandb_register

        logger = logging.getLogger()
        formatter = logging.Formatter("[%(asctime)s] %(message)s")
        handler = logging.StreamHandler(sys.stderr)
        handler.setFormatter(formatter)
        logger.addHandler(handler)


        def main():
            parser = argparse.ArgumentParser(description=None)
            parser.add_argument(
                "-v",
                "--verbose",
                action="count",
                dest="verbosity",
                default=0,
                help="Set verbosity.",
            )
            parser.add_argument("-b", "--api-base", help="What API base url to use.")
            parser.add_argument("-k", "--api-key", help="What API key to use.")
            parser.add_argument(
                "-o",
                "--organization",
                help="Which organization to run as (will use your default organization if not specified)",
            )

            def help(args):
                parser.print_help()

            parser.set_defaults(func=help)

            subparsers = parser.add_subparsers()
            sub_api = subparsers.add_parser("api", help="Direct API calls")
            sub_tools = subparsers.add_parser("tools", help="Client side tools for convenience")
            sub_wandb = subparsers.add_parser("wandb", help="Logging with Weights & Biases")

            api_register(sub_api)
            tools_register(sub_tools)
            wandb_register(sub_wandb)

            args = parser.parse_args()
            if args.verbosity == 1:
                logger.setLevel(logging.INFO)
            elif args.verbosity >= 2:
                logger.setLevel(logging.DEBUG)

            openai.debug = True
            if args.api_key is not None:
                openai.api_key = args.api_key
            if args.api_base is not None:
                openai.api_base = args.api_base
            if args.organization is not None:
                openai.organization = args.organization

            try:
                args.func(args)
            except openai.error.OpenAIError as e:
                display_error(e)
                return 1
            except KeyboardInterrupt:
                sys.stderr.write("\n")
                return 1
            return 0


        if __name__ == "__main__":
            sys.exit(main())

        ############################################################################
        "api_requestor.py  "
            ################################################################
        import asyncio
        import json
        import platform
        import sys
        import threading
        import warnings
        from contextlib import asynccontextmanager
        from json import JSONDecodeError
        from typing import (
            AsyncGenerator,
            AsyncIterator,
            Dict,
            Iterator,
            Optional,
            Tuple,
            Union,
            overload,
        )
        from urllib.parse import urlencode, urlsplit, urlunsplit

        import aiohttp
        import requests

        if sys.version_info >= (3, 8):
            from typing import Literal
        else:
            from typing_extensions import Literal

        import openai
        from openai import error, util, version
        from openai.openai_response import OpenAIResponse
        from openai.util import ApiType

        TIMEOUT_SECS = 600
        MAX_CONNECTION_RETRIES = 2

        # Has one attribute per thread, 'session'.
        _thread_context = threading.local()


        def _build_api_url(url, query):
            scheme, netloc, path, base_query, fragment = urlsplit(url)

            if base_query:
                query = "%s&%s" % (base_query, query)

            return urlunsplit((scheme, netloc, path, query, fragment))


        def _requests_proxies_arg(proxy) -> Optional[Dict[str, str]]:
            """Returns a value suitable for the 'proxies' argument to 'requests.request."""
            if proxy is None:
                return None
            elif isinstance(proxy, str):
                return {"http": proxy, "https": proxy}
            elif isinstance(proxy, dict):
                return proxy.copy()
            else:
                raise ValueError(
                    "'openai.proxy' must be specified as either a string URL or a dict with string URL under the https and/or http keys."
                )


        def _aiohttp_proxies_arg(proxy) -> Optional[str]:
            """Returns a value suitable for the 'proxies' argument to 'aiohttp.ClientSession.request."""
            if proxy is None:
                return None
            elif isinstance(proxy, str):
                return proxy
            elif isinstance(proxy, dict):
                return proxy["https"] if "https" in proxy else proxy["http"]
            else:
                raise ValueError(
                    "'openai.proxy' must be specified as either a string URL or a dict with string URL under the https and/or http keys."
                )


        def _make_session() -> requests.Session:
            if not openai.verify_ssl_certs:
                warnings.warn("verify_ssl_certs is ignored; openai always verifies.")
            s = requests.Session()
            proxies = _requests_proxies_arg(openai.proxy)
            if proxies:
                s.proxies = proxies
            s.mount(
                "https://",
                requests.adapters.HTTPAdapter(max_retries=MAX_CONNECTION_RETRIES),
            )
            return s


        def parse_stream_helper(line: bytes) -> Optional[str]:
            if line:
                if line.strip() == b"data: [DONE]":
                    # return here will cause GeneratorExit exception in urllib3
                    # and it will close http connection with TCP Reset
                    return None
                if line.startswith(b"data: "):
                    line = line[len(b"data: ") :]
                return line.decode("utf-8")
            return None


        def parse_stream(rbody: Iterator[bytes]) -> Iterator[str]:
            for line in rbody:
                _line = parse_stream_helper(line)
                if _line is not None:
                    yield _line


        async def parse_stream_async(rbody: aiohttp.StreamReader):
            async for chunk, _ in rbody.iter_chunks():
                # While the `ChunkTupleAsyncStreamIterator` iterator is meant to iterate over chunks (and thus lines) it seems
                # to still sometimes return multiple lines at a time, so let's split the chunk by lines again.
                for line in chunk.splitlines():
                    _line = parse_stream_helper(line)
                    if _line is not None:
                        yield _line


        class APIRequestor:
            def __init__(
                self,
                key=None,
                api_base=None,
                api_type=None,
                api_version=None,
                organization=None,
            ):
                self.api_base = api_base or openai.api_base
                self.api_key = key or util.default_api_key()
                self.api_type = (
                    ApiType.from_str(api_type)
                    if api_type
                    else ApiType.from_str(openai.api_type)
                )
                self.api_version = api_version or openai.api_version
                self.organization = organization or openai.organization

            @classmethod
            def format_app_info(cls, info):
                str = info["name"]
                if info["version"]:
                    str += "/%s" % (info["version"],)
                if info["url"]:
                    str += " (%s)" % (info["url"],)
                return str

            @overload
            def request(
                self,
                method,
                url,
                params,
                headers,
                files,
                stream: Literal[True],
                request_id: Optional[str] = ...,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = ...,
            ) -> Tuple[Iterator[OpenAIResponse], bool, str]:
                pass

            @overload
            def request(
                self,
                method,
                url,
                params=...,
                headers=...,
                files=...,
                *,
                stream: Literal[True],
                request_id: Optional[str] = ...,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = ...,
            ) -> Tuple[Iterator[OpenAIResponse], bool, str]:
                pass

            @overload
            def request(
                self,
                method,
                url,
                params=...,
                headers=...,
                files=...,
                stream: Literal[False] = ...,
                request_id: Optional[str] = ...,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = ...,
            ) -> Tuple[OpenAIResponse, bool, str]:
                pass

            @overload
            def request(
                self,
                method,
                url,
                params=...,
                headers=...,
                files=...,
                stream: bool = ...,
                request_id: Optional[str] = ...,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = ...,
            ) -> Tuple[Union[OpenAIResponse, Iterator[OpenAIResponse]], bool, str]:
                pass

            def request(
                self,
                method,
                url,
                params=None,
                headers=None,
                files=None,
                stream: bool = False,
                request_id: Optional[str] = None,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = None,
            ) -> Tuple[Union[OpenAIResponse, Iterator[OpenAIResponse]], bool, str]:
                result = self.request_raw(
                    method.lower(),
                    url,
                    params=params,
                    supplied_headers=headers,
                    files=files,
                    stream=stream,
                    request_id=request_id,
                    request_timeout=request_timeout,
                )
                resp, got_stream = self._interpret_response(result, stream)
                return resp, got_stream, self.api_key

            @overload
            async def arequest(
                self,
                method,
                url,
                params,
                headers,
                files,
                stream: Literal[True],
                request_id: Optional[str] = ...,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = ...,
            ) -> Tuple[AsyncGenerator[OpenAIResponse, None], bool, str]:
                pass

            @overload
            async def arequest(
                self,
                method,
                url,
                params=...,
                headers=...,
                files=...,
                *,
                stream: Literal[True],
                request_id: Optional[str] = ...,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = ...,
            ) -> Tuple[AsyncGenerator[OpenAIResponse, None], bool, str]:
                pass

            @overload
            async def arequest(
                self,
                method,
                url,
                params=...,
                headers=...,
                files=...,
                stream: Literal[False] = ...,
                request_id: Optional[str] = ...,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = ...,
            ) -> Tuple[OpenAIResponse, bool, str]:
                pass

            @overload
            async def arequest(
                self,
                method,
                url,
                params=...,
                headers=...,
                files=...,
                stream: bool = ...,
                request_id: Optional[str] = ...,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = ...,
            ) -> Tuple[Union[OpenAIResponse, AsyncGenerator[OpenAIResponse, None]], bool, str]:
                pass

            async def arequest(
                self,
                method,
                url,
                params=None,
                headers=None,
                files=None,
                stream: bool = False,
                request_id: Optional[str] = None,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = None,
            ) -> Tuple[Union[OpenAIResponse, AsyncGenerator[OpenAIResponse, None]], bool, str]:
                ctx = aiohttp_session()
                session = await ctx.__aenter__()
                try:
                    result = await self.arequest_raw(
                        method.lower(),
                        url,
                        session,
                        params=params,
                        supplied_headers=headers,
                        files=files,
                        request_id=request_id,
                        request_timeout=request_timeout,
                    )
                    resp, got_stream = await self._interpret_async_response(result, stream)
                except Exception:
                    await ctx.__aexit__(None, None, None)
                    raise
                if got_stream:

                    async def wrap_resp():
                        assert isinstance(resp, AsyncGenerator)
                        try:
                            async for r in resp:
                                yield r
                        finally:
                            await ctx.__aexit__(None, None, None)

                    return wrap_resp(), got_stream, self.api_key
                else:
                    await ctx.__aexit__(None, None, None)
                    return resp, got_stream, self.api_key

            def handle_error_response(self, rbody, rcode, resp, rheaders, stream_error=False):
                try:
                    error_data = resp["error"]
                except (KeyError, TypeError):
                    raise error.APIError(
                        "Invalid response object from API: %r (HTTP response code "
                        "was %d)" % (rbody, rcode),
                        rbody,
                        rcode,
                        resp,
                    )

                if "internal_message" in error_data:
                    error_data["message"] += "\n\n" + error_data["internal_message"]

                util.log_info(
                    "OpenAI API error received",
                    error_code=error_data.get("code"),
                    error_type=error_data.get("type"),
                    error_message=error_data.get("message"),
                    error_param=error_data.get("param"),
                    stream_error=stream_error,
                )

                # Rate limits were previously coded as 400's with code 'rate_limit'
                if rcode == 429:
                    return error.RateLimitError(
                        error_data.get("message"), rbody, rcode, resp, rheaders
                    )
                elif rcode in [400, 404, 415]:
                    return error.InvalidRequestError(
                        error_data.get("message"),
                        error_data.get("param"),
                        error_data.get("code"),
                        rbody,
                        rcode,
                        resp,
                        rheaders,
                    )
                elif rcode == 401:
                    return error.AuthenticationError(
                        error_data.get("message"), rbody, rcode, resp, rheaders
                    )
                elif rcode == 403:
                    return error.PermissionError(
                        error_data.get("message"), rbody, rcode, resp, rheaders
                    )
                elif rcode == 409:
                    return error.TryAgain(
                        error_data.get("message"), rbody, rcode, resp, rheaders
                    )
                elif stream_error:
                    # TODO: we will soon attach status codes to stream errors
                    parts = [error_data.get("message"), "(Error occurred while streaming.)"]
                    message = " ".join([p for p in parts if p is not None])
                    return error.APIError(message, rbody, rcode, resp, rheaders)
                else:
                    return error.APIError(
                        f"{error_data.get('message')} {rbody} {rcode} {resp} {rheaders}",
                        rbody,
                        rcode,
                        resp,
                        rheaders,
                    )

            def request_headers(
                self, method: str, extra, request_id: Optional[str]
            ) -> Dict[str, str]:
                user_agent = "OpenAI/v1 PythonBindings/%s" % (version.VERSION,)
                if openai.app_info:
                    user_agent += " " + self.format_app_info(openai.app_info)

                uname_without_node = " ".join(
                    v for k, v in platform.uname()._asdict().items() if k != "node"
                )
                ua = {
                    "bindings_version": version.VERSION,
                    "httplib": "requests",
                    "lang": "python",
                    "lang_version": platform.python_version(),
                    "platform": platform.platform(),
                    "publisher": "openai",
                    "uname": uname_without_node,
                }
                if openai.app_info:
                    ua["application"] = openai.app_info

                headers = {
                    "X-OpenAI-Client-User-Agent": json.dumps(ua),
                    "User-Agent": user_agent,
                }

                headers.update(util.api_key_to_header(self.api_type, self.api_key))

                if self.organization:
                    headers["OpenAI-Organization"] = self.organization

                if self.api_version is not None and self.api_type == ApiType.OPEN_AI:
                    headers["OpenAI-Version"] = self.api_version
                if request_id is not None:
                    headers["X-Request-Id"] = request_id
                if openai.debug:
                    headers["OpenAI-Debug"] = "true"
                headers.update(extra)

                return headers

            def _validate_headers(
                self, supplied_headers: Optional[Dict[str, str]]
            ) -> Dict[str, str]:
                headers: Dict[str, str] = {}
                if supplied_headers is None:
                    return headers

                if not isinstance(supplied_headers, dict):
                    raise TypeError("Headers must be a dictionary")

                for k, v in supplied_headers.items():
                    if not isinstance(k, str):
                        raise TypeError("Header keys must be strings")
                    if not isinstance(v, str):
                        raise TypeError("Header values must be strings")
                    headers[k] = v

                # NOTE: It is possible to do more validation of the headers, but a request could always
                # be made to the API manually with invalid headers, so we need to handle them server side.

                return headers

            def _prepare_request_raw(
                self,
                url,
                supplied_headers,
                method,
                params,
                files,
                request_id: Optional[str],
            ) -> Tuple[str, Dict[str, str], Optional[bytes]]:
                abs_url = "%s%s" % (self.api_base, url)
                headers = self._validate_headers(supplied_headers)

                data = None
                if method == "get" or method == "delete":
                    if params:
                        encoded_params = urlencode(
                            [(k, v) for k, v in params.items() if v is not None]
                        )
                        abs_url = _build_api_url(abs_url, encoded_params)
                elif method in {"post", "put"}:
                    if params and files:
                        raise ValueError("At most one of params and files may be specified.")
                    if params:
                        data = json.dumps(params).encode()
                        headers["Content-Type"] = "application/json"
                else:
                    raise error.APIConnectionError(
                        "Unrecognized HTTP method %r. This may indicate a bug in the "
                        "OpenAI bindings. Please contact support@openai.com for "
                        "assistance." % (method,)
                    )

                headers = self.request_headers(method, headers, request_id)

                util.log_debug("Request to OpenAI API", method=method, path=abs_url)
                util.log_debug("Post details", data=data, api_version=self.api_version)

                return abs_url, headers, data

            def request_raw(
                self,
                method,
                url,
                *,
                params=None,
                supplied_headers: Optional[Dict[str, str]] = None,
                files=None,
                stream: bool = False,
                request_id: Optional[str] = None,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = None,
            ) -> requests.Response:
                abs_url, headers, data = self._prepare_request_raw(
                    url, supplied_headers, method, params, files, request_id
                )

                if not hasattr(_thread_context, "session"):
                    _thread_context.session = _make_session()
                try:
                    result = _thread_context.session.request(
                        method,
                        abs_url,
                        headers=headers,
                        data=data,
                        files=files,
                        stream=stream,
                        timeout=request_timeout if request_timeout else TIMEOUT_SECS,
                    )
                except requests.exceptions.Timeout as e:
                    raise error.Timeout("Request timed out: {}".format(e)) from e
                except requests.exceptions.RequestException as e:
                    raise error.APIConnectionError(
                        "Error communicating with OpenAI: {}".format(e)
                    ) from e
                util.log_debug(
                    "OpenAI API response",
                    path=abs_url,
                    response_code=result.status_code,
                    processing_ms=result.headers.get("OpenAI-Processing-Ms"),
                    request_id=result.headers.get("X-Request-Id"),
                )
                # Don't read the whole stream for debug logging unless necessary.
                if openai.log == "debug":
                    util.log_debug(
                        "API response body", body=result.content, headers=result.headers
                    )
                return result

            async def arequest_raw(
                self,
                method,
                url,
                session,
                *,
                params=None,
                supplied_headers: Optional[Dict[str, str]] = None,
                files=None,
                request_id: Optional[str] = None,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = None,
            ) -> aiohttp.ClientResponse:
                abs_url, headers, data = self._prepare_request_raw(
                    url, supplied_headers, method, params, files, request_id
                )

                if isinstance(request_timeout, tuple):
                    timeout = aiohttp.ClientTimeout(
                        connect=request_timeout[0],
                        total=request_timeout[1],
                    )
                else:
                    timeout = aiohttp.ClientTimeout(
                        total=request_timeout if request_timeout else TIMEOUT_SECS
                    )

                if files:
                    # TODO: Use `aiohttp.MultipartWriter` to create the multipart form data here.
                    # For now we use the private `requests` method that is known to have worked so far.
                    data, content_type = requests.models.RequestEncodingMixin._encode_files(  # type: ignore
                        files, data
                    )
                    headers["Content-Type"] = content_type
                request_kwargs = {
                    "method": method,
                    "url": abs_url,
                    "headers": headers,
                    "data": data,
                    "proxy": _aiohttp_proxies_arg(openai.proxy),
                    "timeout": timeout,
                }
                try:
                    result = await session.request(**request_kwargs)
                    util.log_info(
                        "OpenAI API response",
                        path=abs_url,
                        response_code=result.status,
                        processing_ms=result.headers.get("OpenAI-Processing-Ms"),
                        request_id=result.headers.get("X-Request-Id"),
                    )
                    # Don't read the whole stream for debug logging unless necessary.
                    if openai.log == "debug":
                        util.log_debug(
                            "API response body", body=result.content, headers=result.headers
                        )
                    return result
                except (aiohttp.ServerTimeoutError, asyncio.TimeoutError) as e:
                    raise error.Timeout("Request timed out") from e
                except aiohttp.ClientError as e:
                    raise error.APIConnectionError("Error communicating with OpenAI") from e

            def _interpret_response(
                self, result: requests.Response, stream: bool
            ) -> Tuple[Union[OpenAIResponse, Iterator[OpenAIResponse]], bool]:
                """Returns the response(s) and a bool indicating whether it is a stream."""
                if stream and "text/event-stream" in result.headers.get("Content-Type", ""):
                    return (
                        self._interpret_response_line(
                            line, result.status_code, result.headers, stream=True
                        )
                        for line in parse_stream(result.iter_lines())
                    ), True
                else:
                    return (
                        self._interpret_response_line(
                            result.content.decode("utf-8"),
                            result.status_code,
                            result.headers,
                            stream=False,
                        ),
                        False,
                    )

            async def _interpret_async_response(
                self, result: aiohttp.ClientResponse, stream: bool
            ) -> Tuple[Union[OpenAIResponse, AsyncGenerator[OpenAIResponse, None]], bool]:
                """Returns the response(s) and a bool indicating whether it is a stream."""
                if stream and "text/event-stream" in result.headers.get("Content-Type", ""):
                    return (
                        self._interpret_response_line(
                            line, result.status, result.headers, stream=True
                        )
                        async for line in parse_stream_async(result.content)
                    ), True
                else:
                    try:
                        await result.read()
                    except aiohttp.ClientError as e:
                        util.log_warn(e, body=result.content)
                    return (
                        self._interpret_response_line(
                            (await result.read()).decode("utf-8"),
                            result.status,
                            result.headers,
                            stream=False,
                        ),
                        False,
                    )

            def _interpret_response_line(
                self, rbody: str, rcode: int, rheaders, stream: bool
            ) -> OpenAIResponse:
                # HTTP 204 response code does not have any content in the body.
                if rcode == 204:
                    return OpenAIResponse(None, rheaders)

                if rcode == 503:
                    raise error.ServiceUnavailableError(
                        "The server is overloaded or not ready yet.",
                        rbody,
                        rcode,
                        headers=rheaders,
                    )
                try:
                    data = json.loads(rbody)
                except (JSONDecodeError, UnicodeDecodeError) as e:
                    raise error.APIError(
                        f"HTTP code {rcode} from API ({rbody})", rbody, rcode, headers=rheaders
                    ) from e
                resp = OpenAIResponse(data, rheaders)
                # In the future, we might add a "status" parameter to errors
                # to better handle the "error while streaming" case.
                stream_error = stream and "error" in resp.data
                if stream_error or not 200 <= rcode < 300:
                    raise self.handle_error_response(
                        rbody, rcode, resp.data, rheaders, stream_error=stream_error
                    )
                return resp


        @asynccontextmanager
        async def aiohttp_session() -> AsyncIterator[aiohttp.ClientSession]:
            user_set_session = openai.aiosession.get()
            if user_set_session:
                yield user_set_session
            else:
                async with aiohttp.ClientSession() as session:
                    yield session


        ############################################################################
        "cli.py  "
            ################################################################

        import datetime
        import os
        import signal
        import sys
        import warnings
        from typing import Optional

        import requests

        import openai
        from openai.upload_progress import BufferReader
        from openai.validators import (
            apply_necessary_remediation,
            apply_validators,
            get_validators,
            read_any_format,
            write_out_file,
        )


        class bcolors:
            HEADER = "\033[95m"
            OKBLUE = "\033[94m"
            OKGREEN = "\033[92m"
            WARNING = "\033[93m"
            FAIL = "\033[91m"
            ENDC = "\033[0m"
            BOLD = "\033[1m"
            UNDERLINE = "\033[4m"


        def organization_info(obj):
            organization = getattr(obj, "organization", None)
            if organization is not None:
                return "[organization={}] ".format(organization)
            else:
                return ""


        def display(obj):
            sys.stderr.write(organization_info(obj))
            sys.stderr.flush()
            print(obj)


        def display_error(e):
            extra = (
                " (HTTP status code: {})".format(e.http_status)
                if e.http_status is not None
                else ""
            )
            sys.stderr.write(
                "{}{}Error:{} {}{}\n".format(
                    organization_info(e), bcolors.FAIL, bcolors.ENDC, e, extra
                )
            )


        class Engine:
            @classmethod
            def get(cls, args):
                engine = openai.Engine.retrieve(id=args.id)
                display(engine)

            @classmethod
            def update(cls, args):
                engine = openai.Engine.modify(args.id, replicas=args.replicas)
                display(engine)

            @classmethod
            def generate(cls, args):
                warnings.warn(
                    "Engine.generate is deprecated, use Completion.create", DeprecationWarning
                )
                if args.completions and args.completions > 1 and args.stream:
                    raise ValueError("Can't stream multiple completions with openai CLI")

                kwargs = {}
                if args.model is not None:
                    kwargs["model"] = args.model
                resp = openai.Engine(id=args.id).generate(
                    completions=args.completions,
                    context=args.context,
                    length=args.length,
                    stream=args.stream,
                    temperature=args.temperature,
                    top_p=args.top_p,
                    logprobs=args.logprobs,
                    stop=args.stop,
                    **kwargs,
                )
                if not args.stream:
                    resp = [resp]

                for part in resp:
                    completions = len(part["data"])
                    for c_idx, c in enumerate(part["data"]):
                        if completions > 1:
                            sys.stdout.write("===== Completion {} =====\n".format(c_idx))
                        sys.stdout.write("".join(c["text"]))
                        if completions > 1:
                            sys.stdout.write("\n")
                        sys.stdout.flush()

            @classmethod
            def list(cls, args):
                engines = openai.Engine.list()
                display(engines)


        class Completion:
            @classmethod
            def create(cls, args):
                if args.n is not None and args.n > 1 and args.stream:
                    raise ValueError("Can't stream completions with n>1 with the current CLI")

                if args.engine and args.model:
                    warnings.warn(
                        "In most cases, you should not be specifying both engine and model."
                    )

                resp = openai.Completion.create(
                    engine=args.engine,
                    model=args.model,
                    n=args.n,
                    max_tokens=args.max_tokens,
                    logprobs=args.logprobs,
                    prompt=args.prompt,
                    stream=args.stream,
                    temperature=args.temperature,
                    top_p=args.top_p,
                    stop=args.stop,
                    echo=True,
                )
                if not args.stream:
                    resp = [resp]

                for part in resp:
                    choices = part["choices"]
                    for c_idx, c in enumerate(sorted(choices, key=lambda s: s["index"])):
                        if len(choices) > 1:
                            sys.stdout.write("===== Completion {} =====\n".format(c_idx))
                        sys.stdout.write(c["text"])
                        if len(choices) > 1:
                            sys.stdout.write("\n")
                        sys.stdout.flush()


        class Deployment:
            @classmethod
            def get(cls, args):
                resp = openai.Deployment.retrieve(id=args.id)
                print(resp)

            @classmethod
            def delete(cls, args):
                model = openai.Deployment.delete(args.id)
                print(model)

            @classmethod
            def list(cls, args):
                models = openai.Deployment.list()
                print(models)

            @classmethod
            def create(cls, args):
                models = openai.Deployment.create(model=args.model, scale_settings={"scale_type": args.scale_type})
                print(models)


        class Model:
            @classmethod
            def get(cls, args):
                resp = openai.Model.retrieve(id=args.id)
                print(resp)

            @classmethod
            def delete(cls, args):
                model = openai.Model.delete(args.id)
                print(model)

            @classmethod
            def list(cls, args):
                models = openai.Model.list()
                print(models)


        class File:
            @classmethod
            def create(cls, args):
                with open(args.file, "rb") as file_reader:
                    buffer_reader = BufferReader(file_reader.read(), desc="Upload progress")
                resp = openai.File.create(
                    file=buffer_reader,
                    purpose=args.purpose,
                    user_provided_filename=args.file,
                )
                print(resp)

            @classmethod
            def get(cls, args):
                resp = openai.File.retrieve(id=args.id)
                print(resp)

            @classmethod
            def delete(cls, args):
                file = openai.File.delete(args.id)
                print(file)

            @classmethod
            def list(cls, args):
                file = openai.File.list()
                print(file)


        class Image:
            @classmethod
            def create(cls, args):
                resp = openai.Image.create(
                    prompt=args.prompt,
                    size=args.size,
                    n=args.num_images,
                    response_format=args.response_format,
                )
                print(resp)

            @classmethod
            def create_variation(cls, args):
                with open(args.image, "rb") as file_reader:
                    buffer_reader = BufferReader(file_reader.read(), desc="Upload progress")
                resp = openai.Image.create_variation(
                    image=buffer_reader,
                    size=args.size,
                    n=args.num_images,
                    response_format=args.response_format,
                )
                print(resp)

            @classmethod
            def create_edit(cls, args):
                with open(args.image, "rb") as file_reader:
                    image_reader = BufferReader(file_reader.read(), desc="Upload progress")
                mask_reader = None
                if args.mask is not None:
                    with open(args.mask, "rb") as file_reader:
                        mask_reader = BufferReader(file_reader.read(), desc="Upload progress")
                resp = openai.Image.create_edit(
                    image=image_reader,
                    mask=mask_reader,
                    prompt=args.prompt,
                    size=args.size,
                    n=args.num_images,
                    response_format=args.response_format,
                )
                print(resp)


        class FineTune:
            @classmethod
            def list(cls, args):
                resp = openai.FineTune.list()
                print(resp)

            @classmethod
            def _is_url(cls, file: str):
                return file.lower().startswith("http")

            @classmethod
            def _download_file_from_public_url(cls, url: str) -> Optional[bytes]:
                resp = requests.get(url)
                if resp.status_code == 200:
                    return resp.content
                else:
                    return None

            @classmethod
            def _maybe_upload_file(
                cls,
                file: Optional[str] = None,
                content: Optional[bytes] = None,
                user_provided_file: Optional[str] = None,
                check_if_file_exists: bool = True,
            ):
                # Exactly one of `file` or `content` must be provided
                if (file is None) == (content is None):
                    raise ValueError("Exactly one of `file` or `content` must be provided")

                if content is None:
                    assert file is not None
                    with open(file, "rb") as f:
                        content = f.read()

                if check_if_file_exists:
                    bytes = len(content)
                    matching_files = openai.File.find_matching_files(
                        name=user_provided_file or f.name, bytes=bytes, purpose="fine-tune"
                    )
                    if len(matching_files) > 0:
                        file_ids = [f["id"] for f in matching_files]
                        sys.stdout.write(
                            "Found potentially duplicated files with name '{name}', purpose 'fine-tune' and size {size} bytes\n".format(
                                name=os.path.basename(matching_files[0]["filename"]),
                                size=matching_files[0]["bytes"]
                                if "bytes" in matching_files[0]
                                else matching_files[0]["size"],
                            )
                        )
                        sys.stdout.write("\n".join(file_ids))
                        while True:
                            sys.stdout.write(
                                "\nEnter file ID to reuse an already uploaded file, or an empty string to upload this file anyway: "
                            )
                            inp = sys.stdin.readline().strip()
                            if inp in file_ids:
                                sys.stdout.write(
                                    "Reusing already uploaded file: {id}\n".format(id=inp)
                                )
                                return inp
                            elif inp == "":
                                break
                            else:
                                sys.stdout.write(
                                    "File id '{id}' is not among the IDs of the potentially duplicated files\n".format(
                                        id=inp
                                    )
                                )

                buffer_reader = BufferReader(content, desc="Upload progress")
                resp = openai.File.create(
                    file=buffer_reader,
                    purpose="fine-tune",
                    user_provided_filename=user_provided_file or file,
                )
                sys.stdout.write(
                    "Uploaded file from {file}: {id}\n".format(
                        file=user_provided_file or file, id=resp["id"]
                    )
                )
                return resp["id"]

            @classmethod
            def _get_or_upload(cls, file, check_if_file_exists=True):
                try:
                    # 1. If it's a valid file, use it
                    openai.File.retrieve(file)
                    return file
                except openai.error.InvalidRequestError:
                    pass
                if os.path.isfile(file):
                    # 2. If it's a file on the filesystem, upload it
                    return cls._maybe_upload_file(
                        file=file, check_if_file_exists=check_if_file_exists
                    )
                if cls._is_url(file):
                    # 3. If it's a URL, download it temporarily
                    content = cls._download_file_from_public_url(file)
                    if content is not None:
                        return cls._maybe_upload_file(
                            content=content,
                            check_if_file_exists=check_if_file_exists,
                            user_provided_file=file,
                        )
                return file

            @classmethod
            def create(cls, args):
                create_args = {
                    "training_file": cls._get_or_upload(
                        args.training_file, args.check_if_files_exist
                    ),
                }
                if args.validation_file:
                    create_args["validation_file"] = cls._get_or_upload(
                        args.validation_file, args.check_if_files_exist
                    )

                for hparam in (
                    "model",
                    "suffix",
                    "n_epochs",
                    "batch_size",
                    "learning_rate_multiplier",
                    "prompt_loss_weight",
                    "compute_classification_metrics",
                    "classification_n_classes",
                    "classification_positive_class",
                    "classification_betas",
                ):
                    attr = getattr(args, hparam)
                    if attr is not None:
                        create_args[hparam] = attr

                resp = openai.FineTune.create(**create_args)

                if args.no_follow:
                    print(resp)
                    return

                sys.stdout.write(
                    "Created fine-tune: {job_id}\n"
                    "Streaming events until fine-tuning is complete...\n\n"
                    "(Ctrl-C will interrupt the stream, but not cancel the fine-tune)\n".format(
                        job_id=resp["id"]
                    )
                )
                cls._stream_events(resp["id"])

            @classmethod
            def get(cls, args):
                resp = openai.FineTune.retrieve(id=args.id)
                print(resp)

            @classmethod
            def results(cls, args):
                fine_tune = openai.FineTune.retrieve(id=args.id)
                if "result_files" not in fine_tune or len(fine_tune["result_files"]) == 0:
                    raise openai.error.InvalidRequestError(
                        f"No results file available for fine-tune {args.id}", "id"
                    )
                result_file = openai.FineTune.retrieve(id=args.id)["result_files"][0]
                resp = openai.File.download(id=result_file["id"])
                print(resp.decode("utf-8"))

            @classmethod
            def events(cls, args):
                if args.stream:
                    raise openai.error.OpenAIError(
                        message=(
                            "The --stream parameter is deprecated, use fine_tunes.follow "
                            "instead:\n\n"
                            "  openai api fine_tunes.follow -i {id}\n".format(id=args.id)
                        ),
                    )

                resp = openai.FineTune.list_events(id=args.id)  # type: ignore
                print(resp)

            @classmethod
            def follow(cls, args):
                cls._stream_events(args.id)

            @classmethod
            def _stream_events(cls, job_id):
                def signal_handler(sig, frame):
                    status = openai.FineTune.retrieve(job_id).status
                    sys.stdout.write(
                        "\nStream interrupted. Job is still {status}.\n"
                        "To resume the stream, run:\n\n"
                        "  openai api fine_tunes.follow -i {job_id}\n\n"
                        "To cancel your job, run:\n\n"
                        "  openai api fine_tunes.cancel -i {job_id}\n\n".format(
                            status=status, job_id=job_id
                        )
                    )
                    sys.exit(0)

                signal.signal(signal.SIGINT, signal_handler)

                events = openai.FineTune.stream_events(job_id)
                # TODO(rachel): Add a nifty spinner here.
                try:
                    for event in events:
                        sys.stdout.write(
                            "[%s] %s"
                            % (
                                datetime.datetime.fromtimestamp(event["created_at"]),
                                event["message"],
                            )
                        )
                        sys.stdout.write("\n")
                        sys.stdout.flush()
                except Exception:
                    sys.stdout.write(
                        "\nStream interrupted (client disconnected).\n"
                        "To resume the stream, run:\n\n"
                        "  openai api fine_tunes.follow -i {job_id}\n\n".format(job_id=job_id)
                    )
                    return

                resp = openai.FineTune.retrieve(id=job_id)
                status = resp["status"]
                if status == "succeeded":
                    sys.stdout.write("\nJob complete! Status: succeeded 🎉")
                    sys.stdout.write(
                        "\nTry out your fine-tuned model:\n\n"
                        "openai api completions.create -m {model} -p <YOUR_PROMPT>".format(
                            model=resp["fine_tuned_model"]
                        )
                    )
                elif status == "failed":
                    sys.stdout.write(
                        "\nJob failed. Please contact support@openai.com if you need assistance."
                    )
                sys.stdout.write("\n")

            @classmethod
            def cancel(cls, args):
                resp = openai.FineTune.cancel(id=args.id)
                print(resp)

            @classmethod
            def delete(cls, args):
                resp = openai.FineTune.delete(sid=args.id)
                print(resp)

            @classmethod
            def prepare_data(cls, args):

                sys.stdout.write("Analyzing...\n")
                fname = args.file
                auto_accept = args.quiet
                df, remediation = read_any_format(fname)
                apply_necessary_remediation(None, remediation)

                validators = get_validators()

                apply_validators(
                    df,
                    fname,
                    remediation,
                    validators,
                    auto_accept,
                    write_out_file_func=write_out_file,
                )


        class WandbLogger:
            @classmethod
            def sync(cls, args):
                import openai.wandb_logger

                resp = openai.wandb_logger.WandbLogger.sync(
                    id=args.id,
                    n_fine_tunes=args.n_fine_tunes,
                    project=args.project,
                    entity=args.entity,
                    force=args.force,
                )
                print(resp)


        def tools_register(parser):
            subparsers = parser.add_subparsers(
                title="Tools", help="Convenience client side tools"
            )

            def help(args):
                parser.print_help()

            parser.set_defaults(func=help)

            sub = subparsers.add_parser("fine_tunes.prepare_data")
            sub.add_argument(
                "-f",
                "--file",
                required=True,
                help="JSONL, JSON, CSV, TSV, TXT or XLSX file containing prompt-completion examples to be analyzed."
                "This should be the local file path.",
            )
            sub.add_argument(
                "-q",
                "--quiet",
                required=False,
                action="store_true",
                help="Auto accepts all suggestions, without asking for user input. To be used within scripts.",
            )
            sub.set_defaults(func=FineTune.prepare_data)


        def api_register(parser):
            # Engine management
            subparsers = parser.add_subparsers(help="All API subcommands")

            def help(args):
                parser.print_help()

            parser.set_defaults(func=help)

            sub = subparsers.add_parser("engines.list")
            sub.set_defaults(func=Engine.list)

            sub = subparsers.add_parser("engines.get")
            sub.add_argument("-i", "--id", required=True)
            sub.set_defaults(func=Engine.get)

            sub = subparsers.add_parser("engines.update")
            sub.add_argument("-i", "--id", required=True)
            sub.add_argument("-r", "--replicas", type=int)
            sub.set_defaults(func=Engine.update)

            sub = subparsers.add_parser("engines.generate")
            sub.add_argument("-i", "--id", required=True)
            sub.add_argument(
                "--stream", help="Stream tokens as they're ready.", action="store_true"
            )
            sub.add_argument("-c", "--context", help="An optional context to generate from")
            sub.add_argument("-l", "--length", help="How many tokens to generate", type=int)
            sub.add_argument(
                "-t",
                "--temperature",
                help="""What sampling temperature to use. Higher values means the model will take more risks. Try 0.9 for more creative applications, and 0 (argmax sampling) for ones with a well-defined answer.

        Mutually exclusive with `top_p`.""",
                type=float,
            )
            sub.add_argument(
                "-p",
                "--top_p",
                help="""An alternative to sampling with temperature, called nucleus sampling, where the considers the results of the tokens with top_p probability mass. So 0.1 means only the tokens comprising the top 10%% probability mass are considered.

                    Mutually exclusive with `temperature`.""",
                type=float,
            )
            sub.add_argument(
                "-n",
                "--completions",
                help="How many parallel completions to run on this context",
                type=int,
            )
            sub.add_argument(
                "--logprobs",
                help="Include the log probabilites on the `logprobs` most likely tokens. So for example, if `logprobs` is 10, the API will return a list of the 10 most likely tokens. If `logprobs` is supplied, the API will always return the logprob of the generated token, so there may be up to `logprobs+1` elements in the response.",
                type=int,
            )
            sub.add_argument(
                "--stop", help="A stop sequence at which to stop generating tokens."
            )
            sub.add_argument(
                "-m",
                "--model",
                required=False,
                help="A model (most commonly a model ID) to generate from. Defaults to the engine's default model.",
            )
            sub.set_defaults(func=Engine.generate)

            # Completions
            sub = subparsers.add_parser("completions.create")
            sub.add_argument(
                "-e",
                "--engine",
                help="The engine to use. See https://beta.openai.com/docs/engines for more about what engines are available.",
            )
            sub.add_argument(
                "-m",
                "--model",
                help="The model to use. At most one of `engine` or `model` should be specified.",
            )
            sub.add_argument(
                "--stream", help="Stream tokens as they're ready.", action="store_true"
            )
            sub.add_argument("-p", "--prompt", help="An optional prompt to complete from")
            sub.add_argument(
                "-M", "--max-tokens", help="The maximum number of tokens to generate", type=int
            )
            sub.add_argument(
                "-t",
                "--temperature",
                help="""What sampling temperature to use. Higher values means the model will take more risks. Try 0.9 for more creative applications, and 0 (argmax sampling) for ones with a well-defined answer.

        Mutually exclusive with `top_p`.""",
                type=float,
            )
            sub.add_argument(
                "-P",
                "--top_p",
                help="""An alternative to sampling with temperature, called nucleus sampling, where the considers the results of the tokens with top_p probability mass. So 0.1 means only the tokens comprising the top 10%% probability mass are considered.

                    Mutually exclusive with `temperature`.""",
                type=float,
            )
            sub.add_argument(
                "-n",
                "--n",
                help="How many sub-completions to generate for each prompt.",
                type=int,
            )
            sub.add_argument(
                "--logprobs",
                help="Include the log probabilites on the `logprobs` most likely tokens, as well the chosen tokens. So for example, if `logprobs` is 10, the API will return a list of the 10 most likely tokens. If `logprobs` is 0, only the chosen tokens will have logprobs returned.",
                type=int,
            )
            sub.add_argument(
                "--stop", help="A stop sequence at which to stop generating tokens."
            )
            sub.set_defaults(func=Completion.create)

            # Deployments
            sub = subparsers.add_parser("deployments.list")
            sub.set_defaults(func=Deployment.list)

            sub = subparsers.add_parser("deployments.get")
            sub.add_argument("-i", "--id", required=True, help="The deployment ID")
            sub.set_defaults(func=Deployment.get)

            sub = subparsers.add_parser("deployments.delete")
            sub.add_argument("-i", "--id", required=True, help="The deployment ID")
            sub.set_defaults(func=Deployment.delete)
            
            sub = subparsers.add_parser("deployments.create")
            sub.add_argument("-m", "--model", required=True, help="The model ID")
            sub.add_argument("-s", "--scale_type", required=True, help="The scale type. Either 'manual' or 'standard'")
            sub.set_defaults(func=Deployment.create)

            # Models
            sub = subparsers.add_parser("models.list")
            sub.set_defaults(func=Model.list)

            sub = subparsers.add_parser("models.get")
            sub.add_argument("-i", "--id", required=True, help="The model ID")
            sub.set_defaults(func=Model.get)

            sub = subparsers.add_parser("models.delete")
            sub.add_argument("-i", "--id", required=True, help="The model ID")
            sub.set_defaults(func=Model.delete)

            # Files
            sub = subparsers.add_parser("files.create")

            sub.add_argument(
                "-f",
                "--file",
                required=True,
                help="File to upload",
            )
            sub.add_argument(
                "-p",
                "--purpose",
                help="Why are you uploading this file? (see https://beta.openai.com/docs/api-reference/ for purposes)",
                required=True,
            )
            sub.set_defaults(func=File.create)

            sub = subparsers.add_parser("files.get")
            sub.add_argument("-i", "--id", required=True, help="The files ID")
            sub.set_defaults(func=File.get)

            sub = subparsers.add_parser("files.delete")
            sub.add_argument("-i", "--id", required=True, help="The files ID")
            sub.set_defaults(func=File.delete)

            sub = subparsers.add_parser("files.list")
            sub.set_defaults(func=File.list)

            # Finetune
            sub = subparsers.add_parser("fine_tunes.list")
            sub.set_defaults(func=FineTune.list)

            sub = subparsers.add_parser("fine_tunes.create")
            sub.add_argument(
                "-t",
                "--training_file",
                required=True,
                help="JSONL file containing prompt-completion examples for training. This can "
                "be the ID of a file uploaded through the OpenAI API (e.g. file-abcde12345), "
                'a local file path, or a URL that starts with "http".',
            )
            sub.add_argument(
                "-v",
                "--validation_file",
                help="JSONL file containing prompt-completion examples for validation. This can "
                "be the ID of a file uploaded through the OpenAI API (e.g. file-abcde12345), "
                'a local file path, or a URL that starts with "http".',
            )
            sub.add_argument(
                "--no_check_if_files_exist",
                dest="check_if_files_exist",
                action="store_false",
                help="If this argument is set and training_file or validation_file are file paths, immediately upload them. If this argument is not set, check if they may be duplicates of already uploaded files before uploading, based on file name and file size.",
            )
            sub.add_argument(
                "-m",
                "--model",
                help="The model to start fine-tuning from",
            )
            sub.add_argument(
                "--suffix",
                help="If set, this argument can be used to customize the generated fine-tuned model name."
                "All punctuation and whitespace in `suffix` will be replaced with a "
                "single dash, and the string will be lower cased. The max "
                "length of `suffix` is 40 chars. "
                "The generated name will match the form `{base_model}:ft-{org-title}:{suffix}-{timestamp}`. "
                'For example, `openai api fine_tunes.create -t test.jsonl -m ada --suffix "custom model name" '
                "could generate a model with the name "
                "ada:ft-your-org:custom-model-name-2022-02-15-04-21-04",
            )
            sub.add_argument(
                "--no_follow",
                action="store_true",
                help="If set, returns immediately after creating the job. Otherwise, streams events and waits for the job to complete.",
            )
            sub.add_argument(
                "--n_epochs",
                type=int,
                help="The number of epochs to train the model for. An epoch refers to one "
                "full cycle through the training dataset.",
            )
            sub.add_argument(
                "--batch_size",
                type=int,
                help="The batch size to use for training. The batch size is the number of "
                "training examples used to train a single forward and backward pass.",
            )
            sub.add_argument(
                "--learning_rate_multiplier",
                type=float,
                help="The learning rate multiplier to use for training. The fine-tuning "
                "learning rate is determined by the original learning rate used for "
                "pretraining multiplied by this value.",
            )
            sub.add_argument(
                "--prompt_loss_weight",
                type=float,
                help="The weight to use for the prompt loss. The optimum value here depends "
                "depends on your use case. This determines how much the model prioritizes "
                "learning from prompt tokens vs learning from completion tokens.",
            )
            sub.add_argument(
                "--compute_classification_metrics",
                action="store_true",
                help="If set, we calculate classification-specific metrics such as accuracy "
                "and F-1 score using the validation set at the end of every epoch.",
            )
            sub.set_defaults(compute_classification_metrics=None)
            sub.add_argument(
                "--classification_n_classes",
                type=int,
                help="The number of classes in a classification task. This parameter is "
                "required for multiclass classification.",
            )
            sub.add_argument(
                "--classification_positive_class",
                help="The positive class in binary classification. This parameter is needed "
                "to generate precision, recall and F-1 metrics when doing binary "
                "classification.",
            )
            sub.add_argument(
                "--classification_betas",
                type=float,
                nargs="+",
                help="If this is provided, we calculate F-beta scores at the specified beta "
                "values. The F-beta score is a generalization of F-1 score. This is only "
                "used for binary classification.",
            )
            sub.set_defaults(func=FineTune.create)

            sub = subparsers.add_parser("fine_tunes.get")
            sub.add_argument("-i", "--id", required=True, help="The id of the fine-tune job")
            sub.set_defaults(func=FineTune.get)

            sub = subparsers.add_parser("fine_tunes.results")
            sub.add_argument("-i", "--id", required=True, help="The id of the fine-tune job")
            sub.set_defaults(func=FineTune.results)

            sub = subparsers.add_parser("fine_tunes.events")
            sub.add_argument("-i", "--id", required=True, help="The id of the fine-tune job")

            # TODO(rachel): Remove this in 1.0
            sub.add_argument(
                "-s",
                "--stream",
                action="store_true",
                help="[DEPRECATED] If set, events will be streamed until the job is done. Otherwise, "
                "displays the event history to date.",
            )
            sub.set_defaults(func=FineTune.events)

            sub = subparsers.add_parser("fine_tunes.follow")
            sub.add_argument("-i", "--id", required=True, help="The id of the fine-tune job")
            sub.set_defaults(func=FineTune.follow)

            sub = subparsers.add_parser("fine_tunes.cancel")
            sub.add_argument("-i", "--id", required=True, help="The id of the fine-tune job")
            sub.set_defaults(func=FineTune.cancel)

            sub = subparsers.add_parser("fine_tunes.delete")
            sub.add_argument("-i", "--id", required=True, help="The id of the fine-tune job")
            sub.set_defaults(func=FineTune.delete)

            # Image
            sub = subparsers.add_parser("image.create")
            sub.add_argument("-p", "--prompt", type=str, required=True)
            sub.add_argument("-n", "--num-images", type=int, default=1)
            sub.add_argument(
                "-s", "--size", type=str, default="1024x1024", help="Size of the output image"
            )
            sub.add_argument("--response-format", type=str, default="url")
            sub.set_defaults(func=Image.create)

            sub = subparsers.add_parser("image.create_edit")
            sub.add_argument("-p", "--prompt", type=str, required=True)
            sub.add_argument("-n", "--num-images", type=int, default=1)
            sub.add_argument(
                "-I",
                "--image",
                type=str,
                required=True,
                help="Image to modify. Should be a local path and a PNG encoded image.",
            )
            sub.add_argument(
                "-s", "--size", type=str, default="1024x1024", help="Size of the output image"
            )
            sub.add_argument("--response-format", type=str, default="url")
            sub.add_argument(
                "-M",
                "--mask",
                type=str,
                required=False,
                help="Path to a mask image. It should be the same size as the image you're editing and a RGBA PNG image. The Alpha channel acts as the mask.",
            )
            sub.set_defaults(func=Image.create_edit)

            sub = subparsers.add_parser("image.create_variation")
            sub.add_argument("-n", "--num-images", type=int, default=1)
            sub.add_argument(
                "-I",
                "--image",
                type=str,
                required=True,
                help="Image to modify. Should be a local path and a PNG encoded image.",
            )
            sub.add_argument(
                "-s", "--size", type=str, default="1024x1024", help="Size of the output image"
            )
            sub.add_argument("--response-format", type=str, default="url")
            sub.set_defaults(func=Image.create_variation)


        def wandb_register(parser):
            subparsers = parser.add_subparsers(
                title="wandb", help="Logging with Weights & Biases"
            )

            def help(args):
                parser.print_help()

            parser.set_defaults(func=help)

            sub = subparsers.add_parser("sync")
            sub.add_argument("-i", "--id", help="The id of the fine-tune job (optional)")
            sub.add_argument(
                "-n",
                "--n_fine_tunes",
                type=int,
                default=None,
                help="Number of most recent fine-tunes to log when an id is not provided. By default, every fine-tune is synced.",
            )
            sub.add_argument(
                "--project",
                default="GPT-3",
                help="""Name of the project where you're sending runs. By default, it is "GPT-3".""",
            )
            sub.add_argument(
                "--entity",
                help="Username or team name where you're sending runs. By default, your default entity is used, which is usually your username.",
            )
            sub.add_argument(
                "--force",
                action="store_true",
                help="Forces logging and overwrite existing wandb run of the same fine-tune.",
            )
            sub.set_defaults(force=False)
            sub.set_defaults(func=WandbLogger.sync)

        ############################################################################
        "datalib.py "
            ################################################################

        """
        This module helps make data libraries like `numpy` and `pandas` optional dependencies.

        The libraries add up to 130MB+, which makes it challenging to deploy applications
        using this library in environments with code size constraints, like AWS Lambda.

        This module serves as an import proxy and provides a few utilities for dealing with the optionality.

        Since the primary use case of this library (talking to the OpenAI API) doesn’t generally require data libraries,
        it’s safe to make them optional. The rare case when data libraries are needed in the client is handled through
        assertions with instructive error messages.

        See also `setup.py`.

        """
        try:
            import numpy
        except ImportError:
            numpy = None

        try:
            import pandas
        except ImportError:
            pandas = None

        HAS_NUMPY = bool(numpy)
        HAS_PANDAS = bool(pandas)

        INSTRUCTIONS = """

        OpenAI error: 

            missing `{library}` 

        This feature requires additional dependencies:

            $ pip install openai[datalib]

        """

        NUMPY_INSTRUCTIONS = INSTRUCTIONS.format(library="numpy")
        PANDAS_INSTRUCTIONS = INSTRUCTIONS.format(library="pandas")


        class MissingDependencyError(Exception):
            pass


        def assert_has_numpy():
            if not HAS_NUMPY:
                raise MissingDependencyError(NUMPY_INSTRUCTIONS)


        def assert_has_pandas():
            if not HAS_PANDAS:
                raise MissingDependencyError(PANDAS_INSTRUCTIONS)

        ############################################################################
        "embeddings_utils.py "
            ################################################################

        import textwrap as tr
        from typing import List, Optional

        import matplotlib.pyplot as plt
        import plotly.express as px
        from scipy import spatial
        from sklearn.decomposition import PCA
        from sklearn.manifold import TSNE
        from sklearn.metrics import average_precision_score, precision_recall_curve
        from tenacity import retry, stop_after_attempt, wait_random_exponential

        import openai
        from openai.datalib import numpy as np
        from openai.datalib import pandas as pd


        @retry(wait=wait_random_exponential(min=1, max=20), stop=stop_after_attempt(6))
        def get_embedding(text: str, engine="text-similarity-davinci-001") -> List[float]:

            # replace newlines, which can negatively affect performance.
            text = text.replace("\n", " ")

            return openai.Embedding.create(input=[text], engine=engine)["data"][0]["embedding"]


        @retry(wait=wait_random_exponential(min=1, max=20), stop=stop_after_attempt(6))
        async def aget_embedding(
            text: str, engine="text-similarity-davinci-001"
        ) -> List[float]:

            # replace newlines, which can negatively affect performance.
            text = text.replace("\n", " ")

            return (await openai.Embedding.acreate(input=[text], engine=engine))["data"][0][
                "embedding"
            ]


        @retry(wait=wait_random_exponential(min=1, max=20), stop=stop_after_attempt(6))
        def get_embeddings(
            list_of_text: List[str], engine="text-similarity-babbage-001"
        ) -> List[List[float]]:
            assert len(list_of_text) <= 2048, "The batch size should not be larger than 2048."

            # replace newlines, which can negatively affect performance.
            list_of_text = [text.replace("\n", " ") for text in list_of_text]

            data = openai.Embedding.create(input=list_of_text, engine=engine).data
            data = sorted(data, key=lambda x: x["index"])  # maintain the same order as input.
            return [d["embedding"] for d in data]


        @retry(wait=wait_random_exponential(min=1, max=20), stop=stop_after_attempt(6))
        async def aget_embeddings(
            list_of_text: List[str], engine="text-similarity-babbage-001"
        ) -> List[List[float]]:
            assert len(list_of_text) <= 2048, "The batch size should not be larger than 2048."

            # replace newlines, which can negatively affect performance.
            list_of_text = [text.replace("\n", " ") for text in list_of_text]

            data = (await openai.Embedding.acreate(input=list_of_text, engine=engine)).data
            data = sorted(data, key=lambda x: x["index"])  # maintain the same order as input.
            return [d["embedding"] for d in data]


        def cosine_similarity(a, b):
            return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))


        def plot_multiclass_precision_recall(
            y_score, y_true_untransformed, class_list, classifier_name
        ):
            """
            Precision-Recall plotting for a multiclass problem. It plots average precision-recall, per class precision recall and reference f1 contours.

            Code slightly modified, but heavily based on https://scikit-learn.org/stable/auto_examples/model_selection/plot_precision_recall.html
            """
            n_classes = len(class_list)
            y_true = pd.concat(
                [(y_true_untransformed == class_list[i]) for i in range(n_classes)], axis=1
            ).values

            # For each class
            precision = dict()
            recall = dict()
            average_precision = dict()
            for i in range(n_classes):
                precision[i], recall[i], _ = precision_recall_curve(y_true[:, i], y_score[:, i])
                average_precision[i] = average_precision_score(y_true[:, i], y_score[:, i])

            # A "micro-average": quantifying score on all classes jointly
            precision_micro, recall_micro, _ = precision_recall_curve(
                y_true.ravel(), y_score.ravel()
            )
            average_precision_micro = average_precision_score(y_true, y_score, average="micro")
            print(
                str(classifier_name)
                + " - Average precision score over all classes: {0:0.2f}".format(
                    average_precision_micro
                )
            )

            # setup plot details
            plt.figure(figsize=(9, 10))
            f_scores = np.linspace(0.2, 0.8, num=4)
            lines = []
            labels = []
            for f_score in f_scores:
                x = np.linspace(0.01, 1)
                y = f_score * x / (2 * x - f_score)
                (l,) = plt.plot(x[y >= 0], y[y >= 0], color="gray", alpha=0.2)
                plt.annotate("f1={0:0.1f}".format(f_score), xy=(0.9, y[45] + 0.02))

            lines.append(l)
            labels.append("iso-f1 curves")
            (l,) = plt.plot(recall_micro, precision_micro, color="gold", lw=2)
            lines.append(l)
            labels.append(
                "average Precision-recall (auprc = {0:0.2f})" "".format(average_precision_micro)
            )

            for i in range(n_classes):
                (l,) = plt.plot(recall[i], precision[i], lw=2)
                lines.append(l)
                labels.append(
                    "Precision-recall for class `{0}` (auprc = {1:0.2f})"
                    "".format(class_list[i], average_precision[i])
                )

            fig = plt.gcf()
            fig.subplots_adjust(bottom=0.25)
            plt.xlim([0.0, 1.0])
            plt.ylim([0.0, 1.05])
            plt.xlabel("Recall")
            plt.ylabel("Precision")
            plt.title(f"{classifier_name}: Precision-Recall curve for each class")
            plt.legend(lines, labels)


        def distances_from_embeddings(
            query_embedding: List[float],
            embeddings: List[List[float]],
            distance_metric="cosine",
        ) -> List[List]:
            """Return the distances between a query embedding and a list of embeddings."""
            distance_metrics = {
                "cosine": spatial.distance.cosine,
                "L1": spatial.distance.cityblock,
                "L2": spatial.distance.euclidean,
                "Linf": spatial.distance.chebyshev,
            }
            distances = [
                distance_metrics[distance_metric](query_embedding, embedding)
                for embedding in embeddings
            ]
            return distances


        def indices_of_nearest_neighbors_from_distances(distances) -> np.ndarray:
            """Return a list of indices of nearest neighbors from a list of distances."""
            return np.argsort(distances)


        def pca_components_from_embeddings(
            embeddings: List[List[float]], n_components=2
        ) -> np.ndarray:
            """Return the PCA components of a list of embeddings."""
            pca = PCA(n_components=n_components)
            array_of_embeddings = np.array(embeddings)
            return pca.fit_transform(array_of_embeddings)


        def tsne_components_from_embeddings(
            embeddings: List[List[float]], n_components=2, **kwargs
        ) -> np.ndarray:
            """Returns t-SNE components of a list of embeddings."""
            # use better defaults if not specified
            if "init" not in kwargs.keys():
                kwargs["init"] = "pca"
            if "learning_rate" not in kwargs.keys():
                kwargs["learning_rate"] = "auto"
            tsne = TSNE(n_components=n_components, **kwargs)
            array_of_embeddings = np.array(embeddings)
            return tsne.fit_transform(array_of_embeddings)


        def chart_from_components(
            components: np.ndarray,
            labels: Optional[List[str]] = None,
            strings: Optional[List[str]] = None,
            x_title="Component 0",
            y_title="Component 1",
            mark_size=5,
            **kwargs,
        ):
            """Return an interactive 2D chart of embedding components."""
            empty_list = ["" for _ in components]
            data = pd.DataFrame(
                {
                    x_title: components[:, 0],
                    y_title: components[:, 1],
                    "label": labels if labels else empty_list,
                    "string": ["<br>".join(tr.wrap(string, width=30)) for string in strings]
                    if strings
                    else empty_list,
                }
            )
            chart = px.scatter(
                data,
                x=x_title,
                y=y_title,
                color="label" if labels else None,
                symbol="label" if labels else None,
                hover_data=["string"] if strings else None,
                **kwargs,
            ).update_traces(marker=dict(size=mark_size))
            return chart


        def chart_from_components_3D(
            components: np.ndarray,
            labels: Optional[List[str]] = None,
            strings: Optional[List[str]] = None,
            x_title: str = "Component 0",
            y_title: str = "Component 1",
            z_title: str = "Compontent 2",
            mark_size: int = 5,
            **kwargs,
        ):
            """Return an interactive 3D chart of embedding components."""
            empty_list = ["" for _ in components]
            data = pd.DataFrame(
                {
                    x_title: components[:, 0],
                    y_title: components[:, 1],
                    z_title: components[:, 2],
                    "label": labels if labels else empty_list,
                    "string": ["<br>".join(tr.wrap(string, width=30)) for string in strings]
                    if strings
                    else empty_list,
                }
            )
            chart = px.scatter_3d(
                data,
                x=x_title,
                y=y_title,
                z=z_title,
                color="label" if labels else None,
                symbol="label" if labels else None,
                hover_data=["string"] if strings else None,
                **kwargs,
            ).update_traces(marker=dict(size=mark_size))
            return chart

        ############################################################################
        "error.py "
            ################################################################

        import openai


        class OpenAIError(Exception):
            def __init__(
                self,
                message=None,
                http_body=None,
                http_status=None,
                json_body=None,
                headers=None,
                code=None,
            ):
                super(OpenAIError, self).__init__(message)

                if http_body and hasattr(http_body, "decode"):
                    try:
                        http_body = http_body.decode("utf-8")
                    except BaseException:
                        http_body = (
                            "<Could not decode body as utf-8. "
                            "Please report to support@openai.com>"
                        )

                self._message = message
                self.http_body = http_body
                self.http_status = http_status
                self.json_body = json_body
                self.headers = headers or {}
                self.code = code
                self.request_id = self.headers.get("request-id", None)
                self.error = self.construct_error_object()
                self.organization = self.headers.get("openai-organization", None)

            def __str__(self):
                msg = self._message or "<empty message>"
                if self.request_id is not None:
                    return "Request {0}: {1}".format(self.request_id, msg)
                else:
                    return msg

            # Returns the underlying `Exception` (base class) message, which is usually
            # the raw message returned by OpenAI's API. This was previously available
            # in python2 via `error.message`. Unlike `str(error)`, it omits "Request
            # req_..." from the beginning of the string.
            @property
            def user_message(self):
                return self._message

            def __repr__(self):
                return "%s(message=%r, http_status=%r, request_id=%r)" % (
                    self.__class__.__name__,
                    self._message,
                    self.http_status,
                    self.request_id,
                )

            def construct_error_object(self):
                if (
                    self.json_body is None
                    or "error" not in self.json_body
                    or not isinstance(self.json_body["error"], dict)
                ):
                    return None

                return openai.api_resources.error_object.ErrorObject.construct_from(
                    self.json_body["error"]
                )


        class APIError(OpenAIError):
            pass


        class TryAgain(OpenAIError):
            pass


        class Timeout(OpenAIError):
            pass


        class APIConnectionError(OpenAIError):
            def __init__(
                self,
                message,
                http_body=None,
                http_status=None,
                json_body=None,
                headers=None,
                code=None,
                should_retry=False,
            ):
                super(APIConnectionError, self).__init__(
                    message, http_body, http_status, json_body, headers, code
                )
                self.should_retry = should_retry


        class InvalidRequestError(OpenAIError):
            def __init__(
                self,
                message,
                param,
                code=None,
                http_body=None,
                http_status=None,
                json_body=None,
                headers=None,
            ):
                super(InvalidRequestError, self).__init__(
                    message, http_body, http_status, json_body, headers, code
                )
                self.param = param

            def __repr__(self):
                return "%s(message=%r, param=%r, code=%r, http_status=%r, " "request_id=%r)" % (
                    self.__class__.__name__,
                    self._message,
                    self.param,
                    self.code,
                    self.http_status,
                    self.request_id,
                )

            def __reduce__(self):
                return type(self), (
                    self._message,
                    self.param,
                    self.code,
                    self.http_body,
                    self.http_status,
                    self.json_body,
                    self.headers,
                )


        class AuthenticationError(OpenAIError):
            pass


        class PermissionError(OpenAIError):
            pass


        class RateLimitError(OpenAIError):
            pass


        class ServiceUnavailableError(OpenAIError):
            pass


        class InvalidAPIType(OpenAIError):
            pass


        class SignatureVerificationError(OpenAIError):
            def __init__(self, message, sig_header, http_body=None):
                super(SignatureVerificationError, self).__init__(message, http_body)
                self.sig_header = sig_header

            def __reduce__(self):
                return type(self), (
                    self._message,
                    self.sig_header,
                    self.http_body,
                )

        ############################################################################
        "object_classes.py "
            ################################################################

        from openai import api_resources
        from openai.api_resources.experimental.completion_config import CompletionConfig

        OBJECT_CLASSES = {
            "engine": api_resources.Engine,
            "experimental.completion_config": CompletionConfig,
            "file": api_resources.File,
            "fine-tune": api_resources.FineTune,
            "model": api_resources.Model,
            "deployment": api_resources.Deployment,
        }

        ############################################################################
        "openai_object.py  "
            ################################################################

        import json
        from copy import deepcopy
        from typing import Optional, Tuple, Union

        import openai
        from openai import api_requestor, util
        from openai.openai_response import OpenAIResponse
        from openai.util import ApiType


        class OpenAIObject(dict):
            api_base_override = None

            def __init__(
                self,
                id=None,
                api_key=None,
                api_version=None,
                api_type=None,
                organization=None,
                response_ms: Optional[int] = None,
                api_base=None,
                engine=None,
                **params,
            ):
                super(OpenAIObject, self).__init__()

                if response_ms is not None and not isinstance(response_ms, int):
                    raise TypeError(f"response_ms is a {type(response_ms).__name__}.")
                self._response_ms = response_ms

                self._retrieve_params = params

                object.__setattr__(self, "api_key", api_key)
                object.__setattr__(self, "api_version", api_version)
                object.__setattr__(self, "api_type", api_type)
                object.__setattr__(self, "organization", organization)
                object.__setattr__(self, "api_base_override", api_base)
                object.__setattr__(self, "engine", engine)

                if id:
                    self["id"] = id

            @property
            def response_ms(self) -> Optional[int]:
                return self._response_ms

            def __setattr__(self, k, v):
                if k[0] == "_" or k in self.__dict__:
                    return super(OpenAIObject, self).__setattr__(k, v)

                self[k] = v
                return None

            def __getattr__(self, k):
                if k[0] == "_":
                    raise AttributeError(k)
                try:
                    return self[k]
                except KeyError as err:
                    raise AttributeError(*err.args)

            def __delattr__(self, k):
                if k[0] == "_" or k in self.__dict__:
                    return super(OpenAIObject, self).__delattr__(k)
                else:
                    del self[k]

            def __setitem__(self, k, v):
                if v == "":
                    raise ValueError(
                        "You cannot set %s to an empty string. "
                        "We interpret empty strings as None in requests."
                        "You may set %s.%s = None to delete the property" % (k, str(self), k)
                    )
                super(OpenAIObject, self).__setitem__(k, v)

            def __delitem__(self, k):
                raise NotImplementedError("del is not supported")

            # Custom unpickling method that uses `update` to update the dictionary
            # without calling __setitem__, which would fail if any value is an empty
            # string
            def __setstate__(self, state):
                self.update(state)

            # Custom pickling method to ensure the instance is pickled as a custom
            # class and not as a dict, otherwise __setstate__ would not be called when
            # unpickling.
            def __reduce__(self):
                reduce_value = (
                    type(self),  # callable
                    (  # args
                        self.get("id", None),
                        self.api_key,
                        self.api_version,
                        self.api_type,
                        self.organization,
                    ),
                    dict(self),  # state
                )
                return reduce_value

            @classmethod
            def construct_from(
                cls,
                values,
                api_key: Optional[str] = None,
                api_version=None,
                organization=None,
                engine=None,
                response_ms: Optional[int] = None,
            ):
                instance = cls(
                    values.get("id"),
                    api_key=api_key,
                    api_version=api_version,
                    organization=organization,
                    engine=engine,
                    response_ms=response_ms,
                )
                instance.refresh_from(
                    values,
                    api_key=api_key,
                    api_version=api_version,
                    organization=organization,
                    response_ms=response_ms,
                )
                return instance

            def refresh_from(
                self,
                values,
                api_key=None,
                api_version=None,
                api_type=None,
                organization=None,
                response_ms: Optional[int] = None,
            ):
                self.api_key = api_key or getattr(values, "api_key", None)
                self.api_version = api_version or getattr(values, "api_version", None)
                self.api_type = api_type or getattr(values, "api_type", None)
                self.organization = organization or getattr(values, "organization", None)
                self._response_ms = response_ms or getattr(values, "_response_ms", None)

                # Wipe old state before setting new.
                self.clear()
                for k, v in values.items():
                    super(OpenAIObject, self).__setitem__(
                        k, util.convert_to_openai_object(v, api_key, api_version, organization)
                    )

                self._previous = values

            @classmethod
            def api_base(cls):
                return None

            def request(
                self,
                method,
                url,
                params=None,
                headers=None,
                stream=False,
                plain_old_data=False,
                request_id: Optional[str] = None,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = None,
            ):
                if params is None:
                    params = self._retrieve_params
                requestor = api_requestor.APIRequestor(
                    key=self.api_key,
                    api_base=self.api_base_override or self.api_base(),
                    api_type=self.api_type,
                    api_version=self.api_version,
                    organization=self.organization,
                )
                response, stream, api_key = requestor.request(
                    method,
                    url,
                    params=params,
                    stream=stream,
                    headers=headers,
                    request_id=request_id,
                    request_timeout=request_timeout,
                )

                if stream:
                    assert not isinstance(response, OpenAIResponse)  # must be an iterator
                    return (
                        util.convert_to_openai_object(
                            line,
                            api_key,
                            self.api_version,
                            self.organization,
                            plain_old_data=plain_old_data,
                        )
                        for line in response
                    )
                else:
                    return util.convert_to_openai_object(
                        response,
                        api_key,
                        self.api_version,
                        self.organization,
                        plain_old_data=plain_old_data,
                    )

            async def arequest(
                self,
                method,
                url,
                params=None,
                headers=None,
                stream=False,
                plain_old_data=False,
                request_id: Optional[str] = None,
                request_timeout: Optional[Union[float, Tuple[float, float]]] = None,
            ):
                if params is None:
                    params = self._retrieve_params
                requestor = api_requestor.APIRequestor(
                    key=self.api_key,
                    api_base=self.api_base_override or self.api_base(),
                    api_type=self.api_type,
                    api_version=self.api_version,
                    organization=self.organization,
                )
                response, stream, api_key = await requestor.arequest(
                    method,
                    url,
                    params=params,
                    stream=stream,
                    headers=headers,
                    request_id=request_id,
                    request_timeout=request_timeout,
                )

                if stream:
                    assert not isinstance(response, OpenAIResponse)  # must be an iterator
                    return (
                        util.convert_to_openai_object(
                            line,
                            api_key,
                            self.api_version,
                            self.organization,
                            plain_old_data=plain_old_data,
                        )
                        for line in response
                    )
                else:
                    return util.convert_to_openai_object(
                        response,
                        api_key,
                        self.api_version,
                        self.organization,
                        plain_old_data=plain_old_data,
                    )

            def __repr__(self):
                ident_parts = [type(self).__name__]

                obj = self.get("object")
                if isinstance(obj, str):
                    ident_parts.append(obj)

                if isinstance(self.get("id"), str):
                    ident_parts.append("id=%s" % (self.get("id"),))

                unicode_repr = "<%s at %s> JSON: %s" % (
                    " ".join(ident_parts),
                    hex(id(self)),
                    str(self),
                )

                return unicode_repr

            def __str__(self):
                obj = self.to_dict_recursive()
                return json.dumps(obj, sort_keys=True, indent=2)

            def to_dict(self):
                return dict(self)

            def to_dict_recursive(self):
                d = dict(self)
                for k, v in d.items():
                    if isinstance(v, OpenAIObject):
                        d[k] = v.to_dict_recursive()
                    elif isinstance(v, list):
                        d[k] = [
                            e.to_dict_recursive() if isinstance(e, OpenAIObject) else e
                            for e in v
                        ]
                return d

            @property
            def openai_id(self):
                return self.id

            @property
            def typed_api_type(self):
                return (
                    ApiType.from_str(self.api_type)
                    if self.api_type
                    else ApiType.from_str(openai.api_type)
                )

            # This class overrides __setitem__ to throw exceptions on inputs that it
            # doesn't like. This can cause problems when we try to copy an object
            # wholesale because some data that's returned from the API may not be valid
            # if it was set to be set manually. Here we override the class' copy
            # arguments so that we can bypass these possible exceptions on __setitem__.
            def __copy__(self):
                copied = OpenAIObject(
                    self.get("id"),
                    self.api_key,
                    api_version=self.api_version,
                    api_type=self.api_type,
                    organization=self.organization,
                )

                copied._retrieve_params = self._retrieve_params

                for k, v in self.items():
                    # Call parent's __setitem__ to avoid checks that we've added in the
                    # overridden version that can throw exceptions.
                    super(OpenAIObject, copied).__setitem__(k, v)

                return copied

            # This class overrides __setitem__ to throw exceptions on inputs that it
            # doesn't like. This can cause problems when we try to copy an object
            # wholesale because some data that's returned from the API may not be valid
            # if it was set to be set manually. Here we override the class' copy
            # arguments so that we can bypass these possible exceptions on __setitem__.
            def __deepcopy__(self, memo):
                copied = self.__copy__()
                memo[id(self)] = copied

                for k, v in self.items():
                    # Call parent's __setitem__ to avoid checks that we've added in the
                    # overridden version that can throw exceptions.
                    super(OpenAIObject, copied).__setitem__(k, deepcopy(v, memo))

                return copied

        ############################################################################
        "openai_response.py  "
            ################################################################

        from typing import Optional


        class OpenAIResponse:
            def __init__(self, data, headers):
                self._headers = headers
                self.data = data

            @property
            def request_id(self) -> Optional[str]:
                return self._headers.get("request-id")

            @property
            def organization(self) -> Optional[str]:
                return self._headers.get("OpenAI-Organization")

            @property
            def response_ms(self) -> Optional[int]:
                h = self._headers.get("Openai-Processing-Ms")
                return None if h is None else round(float(h))

        ############################################################################
        "upload_progress.py  "
            ################################################################

        import io


        class CancelledError(Exception):
            def __init__(self, msg):
                self.msg = msg
                Exception.__init__(self, msg)

            def __str__(self):
                return self.msg

            __repr__ = __str__


        class BufferReader(io.BytesIO):
            def __init__(self, buf=b"", desc=None):
                self._len = len(buf)
                io.BytesIO.__init__(self, buf)
                self._progress = 0
                self._callback = progress(len(buf), desc=desc)

            def __len__(self):
                return self._len

            def read(self, n=-1):
                chunk = io.BytesIO.read(self, n)
                self._progress += len(chunk)
                if self._callback:
                    try:
                        self._callback(self._progress)
                    except Exception as e:  # catches exception from the callback
                        raise CancelledError("The upload was cancelled: {}".format(e))
                return chunk


        def progress(total, desc):
            import tqdm  # type: ignore

            meter = tqdm.tqdm(total=total, unit_scale=True, desc=desc)

            def incr(progress):
                meter.n = progress
                if progress == total:
                    meter.close()
                else:
                    meter.refresh()

            return incr


        def MB(i):
            return int(i // 1024**2)

        ############################################################################
        "util.py "
            ################################################################

        import logging
        import os
        import re
        import sys
        from enum import Enum
        from typing import Optional

        import openai

        OPENAI_LOG = os.environ.get("OPENAI_LOG")

        logger = logging.getLogger("openai")

        __all__ = [
            "log_info",
            "log_debug",
            "log_warn",
            "logfmt",
        ]

        api_key_to_header = (
            lambda api, key: {"Authorization": f"Bearer {key}"}
            if api in (ApiType.OPEN_AI, ApiType.AZURE_AD)
            else {"api-key": f"{key}"}
        )


        class ApiType(Enum):
            AZURE = 1
            OPEN_AI = 2
            AZURE_AD = 3

            @staticmethod
            def from_str(label):
                if label.lower() == "azure":
                    return ApiType.AZURE
                elif label.lower() in ("azure_ad", "azuread"):
                    return ApiType.AZURE_AD
                elif label.lower() in ("open_ai", "openai"):
                    return ApiType.OPEN_AI
                else:
                    raise openai.error.InvalidAPIType(
                        "The API type provided in invalid. Please select one of the supported API types: 'azure', 'azure_ad', 'open_ai'"
                    )


        def _console_log_level():
            if openai.log in ["debug", "info"]:
                return openai.log
            elif OPENAI_LOG in ["debug", "info"]:
                return OPENAI_LOG
            else:
                return None


        def log_debug(message, **params):
            msg = logfmt(dict(message=message, **params))
            if _console_log_level() == "debug":
                print(msg, file=sys.stderr)
            logger.debug(msg)


        def log_info(message, **params):
            msg = logfmt(dict(message=message, **params))
            if _console_log_level() in ["debug", "info"]:
                print(msg, file=sys.stderr)
            logger.info(msg)


        def log_warn(message, **params):
            msg = logfmt(dict(message=message, **params))
            print(msg, file=sys.stderr)
            logger.warn(msg)


        def logfmt(props):
            def fmt(key, val):
                # Handle case where val is a bytes or bytesarray
                if hasattr(val, "decode"):
                    val = val.decode("utf-8")
                # Check if val is already a string to avoid re-encoding into ascii.
                if not isinstance(val, str):
                    val = str(val)
                if re.search(r"\s", val):
                    val = repr(val)
                # key should already be a string
                if re.search(r"\s", key):
                    key = repr(key)
                return "{key}={val}".format(key=key, val=val)

            return " ".join([fmt(key, val) for key, val in sorted(props.items())])


        def get_object_classes():
            # This is here to avoid a circular dependency
            from openai.object_classes import OBJECT_CLASSES

            return OBJECT_CLASSES


        def convert_to_openai_object(
            resp,
            api_key=None,
            api_version=None,
            organization=None,
            engine=None,
            plain_old_data=False,
        ):
            # If we get a OpenAIResponse, we'll want to return a OpenAIObject.

            response_ms: Optional[int] = None
            if isinstance(resp, openai.openai_response.OpenAIResponse):
                organization = resp.organization
                response_ms = resp.response_ms
                resp = resp.data

            if plain_old_data:
                return resp
            elif isinstance(resp, list):
                return [
                    convert_to_openai_object(
                        i, api_key, api_version, organization, engine=engine
                    )
                    for i in resp
                ]
            elif isinstance(resp, dict) and not isinstance(
                resp, openai.openai_object.OpenAIObject
            ):
                resp = resp.copy()
                klass_name = resp.get("object")
                if isinstance(klass_name, str):
                    klass = get_object_classes().get(
                        klass_name, openai.openai_object.OpenAIObject
                    )
                else:
                    klass = openai.openai_object.OpenAIObject

                return klass.construct_from(
                    resp,
                    api_key=api_key,
                    api_version=api_version,
                    organization=organization,
                    response_ms=response_ms,
                    engine=engine,
                )
            else:
                return resp


        def convert_to_dict(obj):
            """Converts a OpenAIObject back to a regular dict.

            Nested OpenAIObjects are also converted back to regular dicts.

            :param obj: The OpenAIObject to convert.

            :returns: The OpenAIObject as a dict.
            """
            if isinstance(obj, list):
                return [convert_to_dict(i) for i in obj]
            # This works by virtue of the fact that OpenAIObjects _are_ dicts. The dict
            # comprehension returns a regular dict and recursively applies the
            # conversion to each value.
            elif isinstance(obj, dict):
                return {k: convert_to_dict(v) for k, v in obj.items()}
            else:
                return obj


        def merge_dicts(x, y):
            z = x.copy()
            z.update(y)
            return z


        def default_api_key() -> str:
            if openai.api_key_path:
                with open(openai.api_key_path, "rt") as k:
                    api_key = k.read().strip()
                    if not api_key.startswith("sk-"):
                        raise ValueError(f"Malformed API key in {openai.api_key_path}.")
                    return api_key
            elif openai.api_key is not None:
                return openai.api_key
            else:
                raise openai.error.AuthenticationError(
                    "No API key provided. You can set your API key in code using 'openai.api_key = <API-KEY>', or you can set the environment variable OPENAI_API_KEY=<API-KEY>). If your API key is stored in a file, you can point the openai module at it with 'openai.api_key_path = <PATH>'. You can generate API keys in the OpenAI web interface. See https://onboard.openai.com for details, or email support@openai.com if you have any questions."
                )

        ############################################################################
        "validators.py "
            ################################################################

        import os
        import sys
        from typing import Any, Callable, NamedTuple, Optional

        from openai.datalib import pandas as pd, assert_has_pandas


        class Remediation(NamedTuple):
            name: str
            immediate_msg: Optional[str] = None
            necessary_msg: Optional[str] = None
            necessary_fn: Optional[Callable[[Any], Any]] = None
            optional_msg: Optional[str] = None
            optional_fn: Optional[Callable[[Any], Any]] = None
            error_msg: Optional[str] = None


        def num_examples_validator(df):
            """
            This validator will only print out the number of examples and recommend to the user to increase the number of examples if less than 100.
            """
            MIN_EXAMPLES = 100
            optional_suggestion = (
                ""
                if len(df) >= MIN_EXAMPLES
                else ". In general, we recommend having at least a few hundred examples. We've found that performance tends to linearly increase for every doubling of the number of examples"
            )
            immediate_msg = (
                f"\n- Your file contains {len(df)} prompt-completion pairs{optional_suggestion}"
            )
            return Remediation(name="num_examples", immediate_msg=immediate_msg)


        def necessary_column_validator(df, necessary_column):
            """
            This validator will ensure that the necessary column is present in the dataframe.
            """

            def lower_case_column(df, column):
                cols = [c for c in df.columns if c.lower() == column]
                df.rename(columns={cols[0]: column.lower()}, inplace=True)
                return df

            immediate_msg = None
            necessary_fn = None
            necessary_msg = None
            error_msg = None

            if necessary_column not in df.columns:
                if necessary_column in [c.lower() for c in df.columns]:

                    def lower_case_column_creator(df):
                        return lower_case_column(df, necessary_column)

                    necessary_fn = lower_case_column_creator
                    immediate_msg = (
                        f"\n- The `{necessary_column}` column/key should be lowercase"
                    )
                    necessary_msg = f"Lower case column name to `{necessary_column}`"
                else:
                    error_msg = f"`{necessary_column}` column/key is missing. Please make sure you name your columns/keys appropriately, then retry"

            return Remediation(
                name="necessary_column",
                immediate_msg=immediate_msg,
                necessary_msg=necessary_msg,
                necessary_fn=necessary_fn,
                error_msg=error_msg,
            )


        def additional_column_validator(df, fields=["prompt", "completion"]):
            """
            This validator will remove additional columns from the dataframe.
            """
            additional_columns = []
            necessary_msg = None
            immediate_msg = None
            necessary_fn = None
            if len(df.columns) > 2:
                additional_columns = [c for c in df.columns if c not in fields]
                warn_message = ""
                for ac in additional_columns:
                    dups = [c for c in additional_columns if ac in c]
                    if len(dups) > 0:
                        warn_message += f"\n  WARNING: Some of the additional columns/keys contain `{ac}` in their name. These will be ignored, and the column/key `{ac}` will be used instead. This could also result from a duplicate column/key in the provided file."
                immediate_msg = f"\n- The input file should contain exactly two columns/keys per row. Additional columns/keys present are: {additional_columns}{warn_message}"
                necessary_msg = f"Remove additional columns/keys: {additional_columns}"

                def necessary_fn(x):
                    return x[fields]

            return Remediation(
                name="additional_column",
                immediate_msg=immediate_msg,
                necessary_msg=necessary_msg,
                necessary_fn=necessary_fn,
            )


        def non_empty_field_validator(df, field="completion"):
            """
            This validator will ensure that no completion is empty.
            """
            necessary_msg = None
            necessary_fn = None
            immediate_msg = None

            if df[field].apply(lambda x: x == "").any() or df[field].isnull().any():
                empty_rows = (df[field] == "") | (df[field].isnull())
                empty_indexes = df.reset_index().index[empty_rows].tolist()
                immediate_msg = f"\n- `{field}` column/key should not contain empty strings. These are rows: {empty_indexes}"

                def necessary_fn(x):
                    return x[x[field] != ""].dropna(subset=[field])

                necessary_msg = f"Remove {len(empty_indexes)} rows with empty {field}s"
            return Remediation(
                name=f"empty_{field}",
                immediate_msg=immediate_msg,
                necessary_msg=necessary_msg,
                necessary_fn=necessary_fn,
            )


        def duplicated_rows_validator(df, fields=["prompt", "completion"]):
            """
            This validator will suggest to the user to remove duplicate rows if they exist.
            """
            duplicated_rows = df.duplicated(subset=fields)
            duplicated_indexes = df.reset_index().index[duplicated_rows].tolist()
            immediate_msg = None
            optional_msg = None
            optional_fn = None

            if len(duplicated_indexes) > 0:
                immediate_msg = f"\n- There are {len(duplicated_indexes)} duplicated {'-'.join(fields)} sets. These are rows: {duplicated_indexes}"
                optional_msg = f"Remove {len(duplicated_indexes)} duplicate rows"

                def optional_fn(x):
                    return x.drop_duplicates(subset=fields)

            return Remediation(
                name="duplicated_rows",
                immediate_msg=immediate_msg,
                optional_msg=optional_msg,
                optional_fn=optional_fn,
            )


        def long_examples_validator(df):
            """
            This validator will suggest to the user to remove examples that are too long.
            """
            immediate_msg = None
            optional_msg = None
            optional_fn = None

            ft_type = infer_task_type(df)
            if ft_type != "open-ended generation":
                def get_long_indexes(d):
                    long_examples = d.apply(
                        lambda x: len(x.prompt) + len(x.completion) > 10000, axis=1
                    )
                    return d.reset_index().index[long_examples].tolist()

                long_indexes = get_long_indexes(df)

                if len(long_indexes) > 0:
                    immediate_msg = f"\n- There are {len(long_indexes)} examples that are very long. These are rows: {long_indexes}\nFor conditional generation, and for classification the examples shouldn't be longer than 2048 tokens."
                    optional_msg = f"Remove {len(long_indexes)} long examples"

                    def optional_fn(x):
                        
                        long_indexes_to_drop = get_long_indexes(x)
                        if long_indexes != long_indexes_to_drop:
                            sys.stdout.write(f"The indices of the long examples has changed as a result of a previously applied recommendation.\nThe {len(long_indexes_to_drop)} long examples to be dropped are now at the following indices: {long_indexes_to_drop}\n")
                        return x.drop(long_indexes_to_drop)

            return Remediation(
                name="long_examples",
                immediate_msg=immediate_msg,
                optional_msg=optional_msg,
                optional_fn=optional_fn,
            )


        def common_prompt_suffix_validator(df):
            """
            This validator will suggest to add a common suffix to the prompt if one doesn't already exist in case of classification or conditional generation.
            """
            error_msg = None
            immediate_msg = None
            optional_msg = None
            optional_fn = None

            # Find a suffix which is not contained within the prompt otherwise
            suggested_suffix = "\n\n### =>\n\n"
            suffix_options = [
                " ->",
                "\n\n###\n\n",
                "\n\n===\n\n",
                "\n\n---\n\n",
                "\n\n===>\n\n",
                "\n\n--->\n\n",
            ]
            for suffix_option in suffix_options:
                if suffix_option == " ->":
                    if df.prompt.str.contains("\n").any():
                        continue
                if df.prompt.str.contains(suffix_option, regex=False).any():
                    continue
                suggested_suffix = suffix_option
                break
            display_suggested_suffix = suggested_suffix.replace("\n", "\\n")

            ft_type = infer_task_type(df)
            if ft_type == "open-ended generation":
                return Remediation(name="common_suffix")

            def add_suffix(x, suffix):
                x["prompt"] += suffix
                return x

            common_suffix = get_common_xfix(df.prompt, xfix="suffix")
            if (df.prompt == common_suffix).all():
                error_msg = f"All prompts are identical: `{common_suffix}`\nConsider leaving the prompts blank if you want to do open-ended generation, otherwise ensure prompts are different"
                return Remediation(name="common_suffix", error_msg=error_msg)

            if common_suffix != "":
                common_suffix_new_line_handled = common_suffix.replace("\n", "\\n")
                immediate_msg = (
                    f"\n- All prompts end with suffix `{common_suffix_new_line_handled}`"
                )
                if len(common_suffix) > 10:
                    immediate_msg += f". This suffix seems very long. Consider replacing with a shorter suffix, such as `{display_suggested_suffix}`"
                if (
                    df.prompt.str[: -len(common_suffix)]
                    .str.contains(common_suffix, regex=False)
                    .any()
                ):
                    immediate_msg += f"\n  WARNING: Some of your prompts contain the suffix `{common_suffix}` more than once. We strongly suggest that you review your prompts and add a unique suffix"

            else:
                immediate_msg = "\n- Your data does not contain a common separator at the end of your prompts. Having a separator string appended to the end of the prompt makes it clearer to the fine-tuned model where the completion should begin. See https://beta.openai.com/docs/guides/fine-tuning/preparing-your-dataset for more detail and examples. If you intend to do open-ended generation, then you should leave the prompts empty"

            if common_suffix == "":
                optional_msg = (
                    f"Add a suffix separator `{display_suggested_suffix}` to all prompts"
                )

                def optional_fn(x):
                    return add_suffix(x, suggested_suffix)

            return Remediation(
                name="common_completion_suffix",
                immediate_msg=immediate_msg,
                optional_msg=optional_msg,
                optional_fn=optional_fn,
                error_msg=error_msg,
            )


        def common_prompt_prefix_validator(df):
            """
            This validator will suggest to remove a common prefix from the prompt if a long one exist.
            """
            MAX_PREFIX_LEN = 12

            immediate_msg = None
            optional_msg = None
            optional_fn = None

            common_prefix = get_common_xfix(df.prompt, xfix="prefix")
            if common_prefix == "":
                return Remediation(name="common_prefix")

            def remove_common_prefix(x, prefix):
                x["prompt"] = x["prompt"].str[len(prefix) :]
                return x

            if (df.prompt == common_prefix).all():
                # already handled by common_suffix_validator
                return Remediation(name="common_prefix")

            if common_prefix != "":
                immediate_msg = f"\n- All prompts start with prefix `{common_prefix}`"
                if MAX_PREFIX_LEN < len(common_prefix):
                    immediate_msg += ". Fine-tuning doesn't require the instruction specifying the task, or a few-shot example scenario. Most of the time you should only add the input data into the prompt, and the desired output into the completion"
                    optional_msg = f"Remove prefix `{common_prefix}` from all prompts"

                    def optional_fn(x):
                        return remove_common_prefix(x, common_prefix)

            return Remediation(
                name="common_prompt_prefix",
                immediate_msg=immediate_msg,
                optional_msg=optional_msg,
                optional_fn=optional_fn,
            )


        def common_completion_prefix_validator(df):
            """
            This validator will suggest to remove a common prefix from the completion if a long one exist.
            """
            MAX_PREFIX_LEN = 5

            common_prefix = get_common_xfix(df.completion, xfix="prefix")
            ws_prefix = len(common_prefix) > 0 and common_prefix[0] == " "
            if len(common_prefix) < MAX_PREFIX_LEN:
                return Remediation(name="common_prefix")

            def remove_common_prefix(x, prefix, ws_prefix):
                x["completion"] = x["completion"].str[len(prefix) :]
                if ws_prefix:
                    # keep the single whitespace as prefix
                    x["completion"] = " " + x["completion"]
                return x

            if (df.completion == common_prefix).all():
                # already handled by common_suffix_validator
                return Remediation(name="common_prefix")

            immediate_msg = f"\n- All completions start with prefix `{common_prefix}`. Most of the time you should only add the output data into the completion, without any prefix"
            optional_msg = f"Remove prefix `{common_prefix}` from all completions"

            def optional_fn(x):
                return remove_common_prefix(x, common_prefix, ws_prefix)

            return Remediation(
                name="common_completion_prefix",
                immediate_msg=immediate_msg,
                optional_msg=optional_msg,
                optional_fn=optional_fn,
            )


        def common_completion_suffix_validator(df):
            """
            This validator will suggest to add a common suffix to the completion if one doesn't already exist in case of classification or conditional generation.
            """
            error_msg = None
            immediate_msg = None
            optional_msg = None
            optional_fn = None

            ft_type = infer_task_type(df)
            if ft_type == "open-ended generation" or ft_type == "classification":
                return Remediation(name="common_suffix")

            common_suffix = get_common_xfix(df.completion, xfix="suffix")
            if (df.completion == common_suffix).all():
                error_msg = f"All completions are identical: `{common_suffix}`\nEnsure completions are different, otherwise the model will just repeat `{common_suffix}`"
                return Remediation(name="common_suffix", error_msg=error_msg)

            # Find a suffix which is not contained within the completion otherwise
            suggested_suffix = " [END]"
            suffix_options = [
                "\n",
                ".",
                " END",
                "***",
                "+++",
                "&&&",
                "$$$",
                "@@@",
                "%%%",
            ]
            for suffix_option in suffix_options:
                if df.completion.str.contains(suffix_option, regex=False).any():
                    continue
                suggested_suffix = suffix_option
                break
            display_suggested_suffix = suggested_suffix.replace("\n", "\\n")

            def add_suffix(x, suffix):
                x["completion"] += suffix
                return x

            if common_suffix != "":
                common_suffix_new_line_handled = common_suffix.replace("\n", "\\n")
                immediate_msg = (
                    f"\n- All completions end with suffix `{common_suffix_new_line_handled}`"
                )
                if len(common_suffix) > 10:
                    immediate_msg += f". This suffix seems very long. Consider replacing with a shorter suffix, such as `{display_suggested_suffix}`"
                if (
                    df.completion.str[: -len(common_suffix)]
                    .str.contains(common_suffix, regex=False)
                    .any()
                ):
                    immediate_msg += f"\n  WARNING: Some of your completions contain the suffix `{common_suffix}` more than once. We suggest that you review your completions and add a unique ending"

            else:
                immediate_msg = "\n- Your data does not contain a common ending at the end of your completions. Having a common ending string appended to the end of the completion makes it clearer to the fine-tuned model where the completion should end. See https://beta.openai.com/docs/guides/fine-tuning/preparing-your-dataset for more detail and examples."

            if common_suffix == "":
                optional_msg = (
                    f"Add a suffix ending `{display_suggested_suffix}` to all completions"
                )

                def optional_fn(x):
                    return add_suffix(x, suggested_suffix)

            return Remediation(
                name="common_completion_suffix",
                immediate_msg=immediate_msg,
                optional_msg=optional_msg,
                optional_fn=optional_fn,
                error_msg=error_msg,
            )


        def completions_space_start_validator(df):
            """
            This validator will suggest to add a space at the start of the completion if it doesn't already exist. This helps with tokenization.
            """

            def add_space_start(x):
                x["completion"] = x["completion"].apply(
                    lambda x: ("" if x[0] == " " else " ") + x
                )
                return x

            optional_msg = None
            optional_fn = None
            immediate_msg = None

            if df.completion.str[:1].nunique() != 1 or df.completion.values[0][0] != " ":
                immediate_msg = "\n- The completion should start with a whitespace character (` `). This tends to produce better results due to the tokenization we use. See https://beta.openai.com/docs/guides/fine-tuning/preparing-your-dataset for more details"
                optional_msg = "Add a whitespace character to the beginning of the completion"
                optional_fn = add_space_start
            return Remediation(
                name="completion_space_start",
                immediate_msg=immediate_msg,
                optional_msg=optional_msg,
                optional_fn=optional_fn,
            )


        def lower_case_validator(df, column):
            """
            This validator will suggest to lowercase the column values, if more than a third of letters are uppercase.
            """

            def lower_case(x):
                x[column] = x[column].str.lower()
                return x

            count_upper = (
                df[column]
                .apply(lambda x: sum(1 for c in x if c.isalpha() and c.isupper()))
                .sum()
            )
            count_lower = (
                df[column]
                .apply(lambda x: sum(1 for c in x if c.isalpha() and c.islower()))
                .sum()
            )

            if count_upper * 2 > count_lower:
                return Remediation(
                    name="lower_case",
                    immediate_msg=f"\n- More than a third of your `{column}` column/key is uppercase. Uppercase {column}s tends to perform worse than a mixture of case encountered in normal language. We recommend to lower case the data if that makes sense in your domain. See https://beta.openai.com/docs/guides/fine-tuning/preparing-your-dataset for more details",
                    optional_msg=f"Lowercase all your data in column/key `{column}`",
                    optional_fn=lower_case,
                )


        def read_any_format(fname, fields=["prompt", "completion"]):
            """
            This function will read a file saved in .csv, .json, .txt, .xlsx or .tsv format using pandas.
             - for .xlsx it will read the first sheet
             - for .txt it will assume completions and split on newline
            """
            assert_has_pandas()
            remediation = None
            necessary_msg = None
            immediate_msg = None
            error_msg = None
            df = None

            if os.path.isfile(fname):
                for ending, separator in [(".csv", ","), (".tsv", "\t")]:
                    if fname.lower().endswith(ending):
                        immediate_msg = f"\n- Based on your file extension, your file is formatted as a {ending[1:].upper()} file"
                        necessary_msg = (
                            f"Your format `{ending[1:].upper()}` will be converted to `JSONL`"
                        )
                        df = pd.read_csv(fname, sep=separator, dtype=str)
                if fname.lower().endswith(".xlsx"):
                    immediate_msg = "\n- Based on your file extension, your file is formatted as an Excel file"
                    necessary_msg = "Your format `XLSX` will be converted to `JSONL`"
                    xls = pd.ExcelFile(fname)
                    sheets = xls.sheet_names
                    if len(sheets) > 1:
                        immediate_msg += "\n- Your Excel file contains more than one sheet. Please either save as csv or ensure all data is present in the first sheet. WARNING: Reading only the first sheet..."
                    df = pd.read_excel(fname, dtype=str)
                if fname.lower().endswith(".txt"):
                    immediate_msg = "\n- Based on your file extension, you provided a text file"
                    necessary_msg = "Your format `TXT` will be converted to `JSONL`"
                    with open(fname, "r") as f:
                        content = f.read()
                        df = pd.DataFrame(
                            [["", line] for line in content.split("\n")],
                            columns=fields,
                            dtype=str,
                        )
                if fname.lower().endswith("jsonl") or fname.lower().endswith("json"):
                    try:
                        df = pd.read_json(fname, lines=True, dtype=str)
                    except (ValueError, TypeError):
                        df = pd.read_json(fname, dtype=str)
                        immediate_msg = "\n- Your file appears to be in a .JSON format. Your file will be converted to JSONL format"
                        necessary_msg = "Your format `JSON` will be converted to `JSONL`"

                if df is None:
                    error_msg = (
                        "Your file is not saved as a .CSV, .TSV, .XLSX, .TXT or .JSONL file."
                    )
                    if "." in fname:
                        error_msg += (
                            f" Your file `{fname}` appears to end with `.{fname.split('.')[1]}`"
                        )
                    else:
                        error_msg += f" Your file `{fname}` does not appear to have a file ending. Please ensure your filename ends with one of the supported file endings."
                else:
                    df.fillna("", inplace=True)
            else:
                error_msg = f"File {fname} does not exist."

            remediation = Remediation(
                name="read_any_format",
                necessary_msg=necessary_msg,
                immediate_msg=immediate_msg,
                error_msg=error_msg,
            )
            return df, remediation


        def format_inferrer_validator(df):
            """
            This validator will infer the likely fine-tuning format of the data, and display it to the user if it is classification.
            It will also suggest to use ada and explain train/validation split benefits.
            """
            ft_type = infer_task_type(df)
            immediate_msg = None
            if ft_type == "classification":
                immediate_msg = f"\n- Based on your data it seems like you're trying to fine-tune a model for {ft_type}\n- For classification, we recommend you try one of the faster and cheaper models, such as `ada`\n- For classification, you can estimate the expected model performance by keeping a held out dataset, which is not used for training"
            return Remediation(name="num_examples", immediate_msg=immediate_msg)


        def apply_necessary_remediation(df, remediation):
            """
            This function will apply a necessary remediation to a dataframe, or print an error message if one exists.
            """
            if remediation.error_msg is not None:
                sys.stderr.write(
                    f"\n\nERROR in {remediation.name} validator: {remediation.error_msg}\n\nAborting..."
                )
                sys.exit(1)
            if remediation.immediate_msg is not None:
                sys.stdout.write(remediation.immediate_msg)
            if remediation.necessary_fn is not None:
                df = remediation.necessary_fn(df)
            return df


        def accept_suggestion(input_text, auto_accept):
            sys.stdout.write(input_text)
            if auto_accept:
                sys.stdout.write("Y\n")
                return True
            return input().lower() != "n"


        def apply_optional_remediation(df, remediation, auto_accept):
            """
            This function will apply an optional remediation to a dataframe, based on the user input.
            """
            optional_applied = False
            input_text = f"- [Recommended] {remediation.optional_msg} [Y/n]: "
            if remediation.optional_msg is not None:
                if accept_suggestion(input_text, auto_accept):
                    df = remediation.optional_fn(df)
                    optional_applied = True
            if remediation.necessary_msg is not None:
                sys.stdout.write(f"- [Necessary] {remediation.necessary_msg}\n")
            return df, optional_applied


        def estimate_fine_tuning_time(df):
            """
            Estimate the time it'll take to fine-tune the dataset
            """
            ft_format = infer_task_type(df)
            expected_time = 1.0
            if ft_format == "classification":
                num_examples = len(df)
                expected_time = num_examples * 1.44
            else:
                size = df.memory_usage(index=True).sum()
                expected_time = size * 0.0515

            def format_time(time):
                if time < 60:
                    return f"{round(time, 2)} seconds"
                elif time < 3600:
                    return f"{round(time / 60, 2)} minutes"
                elif time < 86400:
                    return f"{round(time / 3600, 2)} hours"
                else:
                    return f"{round(time / 86400, 2)} days"

            time_string = format_time(expected_time + 140)
            sys.stdout.write(
                f"Once your model starts training, it'll approximately take {time_string} to train a `curie` model, and less for `ada` and `babbage`. Queue will approximately take half an hour per job ahead of you.\n"
            )


        def get_outfnames(fname, split):
            suffixes = ["_train", "_valid"] if split else [""]
            i = 0
            while True:
                index_suffix = f" ({i})" if i > 0 else ""
                candidate_fnames = [
                    os.path.splitext(fname)[0] + "_prepared" + suffix + index_suffix + ".jsonl"
                    for suffix in suffixes
                ]
                if not any(os.path.isfile(f) for f in candidate_fnames):
                    return candidate_fnames
                i += 1


        def get_classification_hyperparams(df):
            n_classes = df.completion.nunique()
            pos_class = None
            if n_classes == 2:
                pos_class = df.completion.value_counts().index[0]
            return n_classes, pos_class


        def write_out_file(df, fname, any_remediations, auto_accept):
            """
            This function will write out a dataframe to a file, if the user would like to proceed, and also offer a fine-tuning command with the newly created file.
            For classification it will optionally ask the user if they would like to split the data into train/valid files, and modify the suggested command to include the valid set.
            """
            ft_format = infer_task_type(df)
            common_prompt_suffix = get_common_xfix(df.prompt, xfix="suffix")
            common_completion_suffix = get_common_xfix(df.completion, xfix="suffix")

            split = False
            input_text = "- [Recommended] Would you like to split into training and validation set? [Y/n]: "
            if ft_format == "classification":
                if accept_suggestion(input_text, auto_accept):
                    split = True

            additional_params = ""
            common_prompt_suffix_new_line_handled = common_prompt_suffix.replace("\n", "\\n")
            common_completion_suffix_new_line_handled = common_completion_suffix.replace(
                "\n", "\\n"
            )
            optional_ending_string = (
                f' Make sure to include `stop=["{common_completion_suffix_new_line_handled}"]` so that the generated texts ends at the expected place.'
                if len(common_completion_suffix_new_line_handled) > 0
                else ""
            )

            input_text = "\n\nYour data will be written to a new JSONL file. Proceed [Y/n]: "

            if not any_remediations and not split:
                sys.stdout.write(
                    f'\nYou can use your file for fine-tuning:\n> openai api fine_tunes.create -t "{fname}"{additional_params}\n\nAfter you’ve fine-tuned a model, remember that your prompt has to end with the indicator string `{common_prompt_suffix_new_line_handled}` for the model to start generating completions, rather than continuing with the prompt.{optional_ending_string}\n'
                )
                estimate_fine_tuning_time(df)

            elif accept_suggestion(input_text, auto_accept):
                fnames = get_outfnames(fname, split)
                if split:
                    assert len(fnames) == 2 and "train" in fnames[0] and "valid" in fnames[1]
                    MAX_VALID_EXAMPLES = 1000
                    n_train = max(len(df) - MAX_VALID_EXAMPLES, int(len(df) * 0.8))
                    df_train = df.sample(n=n_train, random_state=42)
                    df_valid = df.drop(df_train.index)
                    df_train[["prompt", "completion"]].to_json(
                        fnames[0], lines=True, orient="records", force_ascii=False
                    )
                    df_valid[["prompt", "completion"]].to_json(
                        fnames[1], lines=True, orient="records", force_ascii=False
                    )

                    n_classes, pos_class = get_classification_hyperparams(df)
                    additional_params += " --compute_classification_metrics"
                    if n_classes == 2:
                        additional_params += f' --classification_positive_class "{pos_class}"'
                    else:
                        additional_params += f" --classification_n_classes {n_classes}"
                else:
                    assert len(fnames) == 1
                    df[["prompt", "completion"]].to_json(
                        fnames[0], lines=True, orient="records", force_ascii=False
                    )

                # Add -v VALID_FILE if we split the file into train / valid
                files_string = ("s" if split else "") + " to `" + ("` and `".join(fnames))
                valid_string = f' -v "{fnames[1]}"' if split else ""
                separator_reminder = (
                    ""
                    if len(common_prompt_suffix_new_line_handled) == 0
                    else f"After you’ve fine-tuned a model, remember that your prompt has to end with the indicator string `{common_prompt_suffix_new_line_handled}` for the model to start generating completions, rather than continuing with the prompt."
                )
                sys.stdout.write(
                    f'\nWrote modified file{files_string}`\nFeel free to take a look!\n\nNow use that file when fine-tuning:\n> openai api fine_tunes.create -t "{fnames[0]}"{valid_string}{additional_params}\n\n{separator_reminder}{optional_ending_string}\n'
                )
                estimate_fine_tuning_time(df)
            else:
                sys.stdout.write("Aborting... did not write the file\n")


        def infer_task_type(df):
            """
            Infer the likely fine-tuning task type from the data
            """
            CLASSIFICATION_THRESHOLD = 3  # min_average instances of each class
            if sum(df.prompt.str.len()) == 0:
                return "open-ended generation"

            if len(df.completion.unique()) < len(df) / CLASSIFICATION_THRESHOLD:
                return "classification"

            return "conditional generation"


        def get_common_xfix(series, xfix="suffix"):
            """
            Finds the longest common suffix or prefix of all the values in a series
            """
            common_xfix = ""
            while True:
                common_xfixes = (
                    series.str[-(len(common_xfix) + 1) :]
                    if xfix == "suffix"
                    else series.str[: len(common_xfix) + 1]
                )  # first few or last few characters
                if (
                    common_xfixes.nunique() != 1
                ):  # we found the character at which we don't have a unique xfix anymore
                    break
                elif (
                    common_xfix == common_xfixes.values[0]
                ):  # the entire first row is a prefix of every other row
                    break
                else:  # the first or last few characters are still common across all rows - let's try to add one more
                    common_xfix = common_xfixes.values[0]
            return common_xfix


        def get_validators():
            return [
                num_examples_validator,
                lambda x: necessary_column_validator(x, "prompt"),
                lambda x: necessary_column_validator(x, "completion"),
                additional_column_validator,
                non_empty_field_validator,
                format_inferrer_validator,
                duplicated_rows_validator,
                long_examples_validator,
                lambda x: lower_case_validator(x, "prompt"),
                lambda x: lower_case_validator(x, "completion"),
                common_prompt_suffix_validator,
                common_prompt_prefix_validator,
                common_completion_prefix_validator,
                common_completion_suffix_validator,
                completions_space_start_validator,
            ]


        def apply_validators(
            df,
            fname,
            remediation,
            validators,
            auto_accept,
            write_out_file_func,
        ):
            optional_remediations = []
            if remediation is not None:
                optional_remediations.append(remediation)
            for validator in validators:
                remediation = validator(df)
                if remediation is not None:
                    optional_remediations.append(remediation)
                    df = apply_necessary_remediation(df, remediation)

            any_optional_or_necessary_remediations = any(
                [
                    remediation
                    for remediation in optional_remediations
                    if remediation.optional_msg is not None
                    or remediation.necessary_msg is not None
                ]
            )
            any_necessary_applied = any(
                [
                    remediation
                    for remediation in optional_remediations
                    if remediation.necessary_msg is not None
                ]
            )
            any_optional_applied = False

            if any_optional_or_necessary_remediations:
                sys.stdout.write(
                    "\n\nBased on the analysis we will perform the following actions:\n"
                )
                for remediation in optional_remediations:
                    df, optional_applied = apply_optional_remediation(
                        df, remediation, auto_accept
                    )
                    any_optional_applied = any_optional_applied or optional_applied
            else:
                sys.stdout.write("\n\nNo remediations found.\n")

            any_optional_or_necessary_applied = any_optional_applied or any_necessary_applied

            write_out_file_func(df, fname, any_optional_or_necessary_applied, auto_accept)

        ############################################################################
        "version.py "
            ################################################################

        VERSION = "0.26.1"

        ############################################################################
        "wandb_logger.py"
            ################################################################

        try:
            import wandb

            WANDB_AVAILABLE = True
        except:
            WANDB_AVAILABLE = False


        if WANDB_AVAILABLE:
            import datetime
            import io
            import json
            import re
            from pathlib import Path

            from openai import File, FineTune
            from openai.datalib import numpy as np
            from openai.datalib import pandas as pd


        class WandbLogger:
            """
            Log fine-tunes to [Weights & Biases](https://wandb.me/openai-docs)
            """

            if not WANDB_AVAILABLE:
                print("Logging requires wandb to be installed. Run `pip install wandb`.")
            else:
                _wandb_api = None
                _logged_in = False

            @classmethod
            def sync(
                cls,
                id=None,
                n_fine_tunes=None,
                project="GPT-3",
                entity=None,
                force=False,
                **kwargs_wandb_init,
            ):
                """
                Sync fine-tunes to Weights & Biases.
                :param id: The id of the fine-tune (optional)
                :param n_fine_tunes: Number of most recent fine-tunes to log when an id is not provided. By default, every fine-tune is synced.
                :param project: Name of the project where you're sending runs. By default, it is "GPT-3".
                :param entity: Username or team name where you're sending runs. By default, your default entity is used, which is usually your username.
                :param force: Forces logging and overwrite existing wandb run of the same fine-tune.
                """

                if not WANDB_AVAILABLE:
                    return

                if id:
                    fine_tune = FineTune.retrieve(id=id)
                    fine_tune.pop("events", None)
                    fine_tunes = [fine_tune]

                else:
                    # get list of fine_tune to log
                    fine_tunes = FineTune.list()
                    if not fine_tunes or fine_tunes.get("data") is None:
                        print("No fine-tune has been retrieved")
                        return
                    fine_tunes = fine_tunes["data"][
                        -n_fine_tunes if n_fine_tunes is not None else None :
                    ]

                # log starting from oldest fine_tune
                show_individual_warnings = (
                    False if id is None and n_fine_tunes is None else True
                )
                fine_tune_logged = [
                    cls._log_fine_tune(
                        fine_tune,
                        project,
                        entity,
                        force,
                        show_individual_warnings,
                        **kwargs_wandb_init,
                    )
                    for fine_tune in fine_tunes
                ]

                if not show_individual_warnings and not any(fine_tune_logged):
                    print("No new successful fine-tunes were found")

                return "🎉 wandb sync completed successfully"

            @classmethod
            def _log_fine_tune(
                cls,
                fine_tune,
                project,
                entity,
                force,
                show_individual_warnings,
                **kwargs_wandb_init,
            ):
                fine_tune_id = fine_tune.get("id")
                status = fine_tune.get("status")

                # check run completed successfully
                if status != "succeeded":
                    if show_individual_warnings:
                        print(
                            f'Fine-tune {fine_tune_id} has the status "{status}" and will not be logged'
                        )
                    return

                # check results are present
                try:
                    results_id = fine_tune["result_files"][0]["id"]
                    results = File.download(id=results_id).decode("utf-8")
                except:
                    if show_individual_warnings:
                        print(f"Fine-tune {fine_tune_id} has no results and will not be logged")
                    return

                # check run has not been logged already
                run_path = f"{project}/{fine_tune_id}"
                if entity is not None:
                    run_path = f"{entity}/{run_path}"
                wandb_run = cls._get_wandb_run(run_path)
                if wandb_run:
                    wandb_status = wandb_run.summary.get("status")
                    if show_individual_warnings:
                        if wandb_status == "succeeded":
                            print(
                                f"Fine-tune {fine_tune_id} has already been logged successfully at {wandb_run.url}"
                            )
                            if not force:
                                print(
                                    'Use "--force" in the CLI or "force=True" in python if you want to overwrite previous run'
                                )
                        else:
                            print(
                                f"A run for fine-tune {fine_tune_id} was previously created but didn't end successfully"
                            )
                        if wandb_status != "succeeded" or force:
                            print(
                                f"A new wandb run will be created for fine-tune {fine_tune_id} and previous run will be overwritten"
                            )
                    if wandb_status == "succeeded" and not force:
                        return

                # start a wandb run
                wandb.init(
                    job_type="fine-tune",
                    config=cls._get_config(fine_tune),
                    project=project,
                    entity=entity,
                    name=fine_tune_id,
                    id=fine_tune_id,
                    **kwargs_wandb_init,
                )

                # log results
                df_results = pd.read_csv(io.StringIO(results))
                for _, row in df_results.iterrows():
                    metrics = {k: v for k, v in row.items() if not np.isnan(v)}
                    step = metrics.pop("step")
                    if step is not None:
                        step = int(step)
                    wandb.log(metrics, step=step)
                fine_tuned_model = fine_tune.get("fine_tuned_model")
                if fine_tuned_model is not None:
                    wandb.summary["fine_tuned_model"] = fine_tuned_model

                # training/validation files and fine-tune details
                cls._log_artifacts(fine_tune, project, entity)

                # mark run as complete
                wandb.summary["status"] = "succeeded"

                wandb.finish()
                return True

            @classmethod
            def _ensure_logged_in(cls):
                if not cls._logged_in:
                    if wandb.login():
                        cls._logged_in = True
                    else:
                        raise Exception("You need to log in to wandb")

            @classmethod
            def _get_wandb_run(cls, run_path):
                cls._ensure_logged_in()
                try:
                    if cls._wandb_api is None:
                        cls._wandb_api = wandb.Api()
                    return cls._wandb_api.run(run_path)
                except Exception:
                    return None

            @classmethod
            def _get_wandb_artifact(cls, artifact_path):
                cls._ensure_logged_in()
                try:
                    if cls._wandb_api is None:
                        cls._wandb_api = wandb.Api()
                    return cls._wandb_api.artifact(artifact_path)
                except Exception:
                    return None

            @classmethod
            def _get_config(cls, fine_tune):
                config = dict(fine_tune)
                for key in ("training_files", "validation_files", "result_files"):
                    if config.get(key) and len(config[key]):
                        config[key] = config[key][0]
                if config.get("created_at"):
                    config["created_at"] = datetime.datetime.fromtimestamp(config["created_at"])
                return config

            @classmethod
            def _log_artifacts(cls, fine_tune, project, entity):
                # training/validation files
                training_file = (
                    fine_tune["training_files"][0]
                    if fine_tune.get("training_files") and len(fine_tune["training_files"])
                    else None
                )
                validation_file = (
                    fine_tune["validation_files"][0]
                    if fine_tune.get("validation_files") and len(fine_tune["validation_files"])
                    else None
                )
                for file, prefix, artifact_type in (
                    (training_file, "train", "training_files"),
                    (validation_file, "valid", "validation_files"),
                ):
                    if file is not None:
                        cls._log_artifact_inputs(file, prefix, artifact_type, project, entity)

                # fine-tune details
                fine_tune_id = fine_tune.get("id")
                artifact = wandb.Artifact(
                    "fine_tune_details",
                    type="fine_tune_details",
                    metadata=fine_tune,
                )
                with artifact.new_file(
                    "fine_tune_details.json", mode="w", encoding="utf-8"
                ) as f:
                    json.dump(fine_tune, f, indent=2)
                wandb.run.log_artifact(
                    artifact,
                    aliases=["latest", fine_tune_id],
                )

            @classmethod
            def _log_artifact_inputs(cls, file, prefix, artifact_type, project, entity):
                file_id = file["id"]
                filename = Path(file["filename"]).name
                stem = Path(file["filename"]).stem

                # get input artifact
                artifact_name = f"{prefix}-{filename}"
                # sanitize name to valid wandb artifact name
                artifact_name = re.sub(r"[^a-zA-Z0-9_\-.]", "_", artifact_name)
                artifact_alias = file_id
                artifact_path = f"{project}/{artifact_name}:{artifact_alias}"
                if entity is not None:
                    artifact_path = f"{entity}/{artifact_path}"
                artifact = cls._get_wandb_artifact(artifact_path)

                # create artifact if file not already logged previously
                if artifact is None:
                    # get file content
                    try:
                        file_content = File.download(id=file_id).decode("utf-8")
                    except:
                        print(
                            f"File {file_id} could not be retrieved. Make sure you are allowed to download training/validation files"
                        )
                        return
                    artifact = wandb.Artifact(artifact_name, type=artifact_type, metadata=file)
                    with artifact.new_file(filename, mode="w", encoding="utf-8") as f:
                        f.write(file_content)

                    # create a Table
                    try:
                        table, n_items = cls._make_table(file_content)
                        artifact.add(table, stem)
                        wandb.config.update({f"n_{prefix}": n_items})
                        artifact.metadata["items"] = n_items
                    except:
                        print(f"File {file_id} could not be read as a valid JSON file")
                else:
                    # log number of items
                    wandb.config.update({f"n_{prefix}": artifact.metadata.get("items")})

                wandb.run.use_artifact(artifact, aliases=["latest", artifact_alias])

            @classmethod
            def _make_table(cls, file_content):
                df = pd.read_json(io.StringIO(file_content), orient="records", lines=True)
                return wandb.Table(dataframe=df), len(df)

        ############################################################################
        "setup.py "
            ################################################################


        import os

        from setuptools import find_packages, setup

        version_contents = {}
        version_path = os.path.join(
            os.path.abspath(os.path.dirname(__file__)), "openai/version.py"
        )
        with open(version_path, "rt") as f:
            exec(f.read(), version_contents)
            
        with open("README.md", "r") as fh:
            long_description = fh.read()


        DATA_LIBRARIES = [
            # These libraries are optional because of their size. See `openai/datalib.py`.
            "numpy",
            "pandas>=1.2.3",  # Needed for CLI fine-tuning data preparation tool
            "pandas-stubs>=1.1.0.11",  # Needed for type hints for mypy
            "openpyxl>=3.0.7",  # Needed for CLI fine-tuning data preparation tool xlsx format
        ]

        setup(
            name="openai",
            description="Python client library for the OpenAI API",
            long_description=long_description,
            long_description_content_type="text/markdown",
            version=version_contents["VERSION"],
            install_requires=[
                "requests>=2.20",  # to get the patch for CVE-2018-18074
                "tqdm",  # Needed for progress bars
                'typing_extensions;python_version<"3.8"',  # Needed for type hints for mypy
                "aiohttp",  # Needed for async support
            ],
            extras_require={
                "dev": ["black~=21.6b0", "pytest==6.*", "pytest-asyncio", "pytest-mock"],
                "datalib": DATA_LIBRARIES,
                "wandb": [
                    "wandb",
                    *DATA_LIBRARIES,
                ],
                "embeddings": [
                    "scikit-learn>=1.0.2",  # Needed for embedding utils, versions >= 1.1 require python 3.8
                    "tenacity>=8.0.1",
                    "matplotlib",
                    "sklearn",
                    "plotly",
                    *DATA_LIBRARIES,
                ],
            },
            python_requires=">=3.7.1",
            entry_points={
                "console_scripts": [
                    "openai=openai._openai_scripts:main",
                ],
            },
            packages=find_packages(exclude=["tests", "tests.*"]),
            package_data={
                "openai": [
                    "py.typed",
                ]
            },
            author="OpenAI",
            author_email="support@openai.com",
            url="https://github.com/openai/openai-python",
        )

        ############################################################################
        ".gitignore "
            ################################################################

        """*egg-info
        idea
        python-version
        /public/dist
        __pycache__
        build
        *egg
        vscode/settings.json
        ipynb_checkpoints
        .vscode/launch.json
        examples/azure/training.jsonl
        examples/azure/validation.jsonl"""

        ############################################################################
        "Makefile "
            ################################################################

        """build:
                python setup.py sdist

        upload:
                twine upload dist/openai-*.tar.gz
                rm dist/openai-*.tar.gz"""

        ############################################################################
        "pyproject.toml "
            ################################################################

        """[tool.black]
        target-version = ['py36']
        exclude = '.*\.ipynb'

        [tool.isort]
        py_version = 36
        include_trailing_comma = "true"
        line_length = 88
        multi_line_output = 3"""

        ############################################################################
        "setup.py "
            ################################################################

        import os

        from setuptools import find_packages, setup

        version_contents = {}
        version_path = os.path.join(
            os.path.abspath(os.path.dirname(__file__)), "openai/version.py"
        )
        with open(version_path, "rt") as f:
            exec(f.read(), version_contents)
            
        with open("README.md", "r") as fh:
            long_description = fh.read()


        DATA_LIBRARIES = [
            # These libraries are optional because of their size. See `openai/datalib.py`.
            "numpy",
            "pandas>=1.2.3",  # Needed for CLI fine-tuning data preparation tool
            "pandas-stubs>=1.1.0.11",  # Needed for type hints for mypy
            "openpyxl>=3.0.7",  # Needed for CLI fine-tuning data preparation tool xlsx format
        ]

        setup(
            name="openai",
            description="Python client library for the OpenAI API",
            long_description=long_description,
            long_description_content_type="text/markdown",
            version=version_contents["VERSION"],
            install_requires=[
                "requests>=2.20",  # to get the patch for CVE-2018-18074
                "tqdm",  # Needed for progress bars
                'typing_extensions;python_version<"3.8"',  # Needed for type hints for mypy
                "aiohttp",  # Needed for async support
            ],
            extras_require={
                "dev": ["black~=21.6b0", "pytest==6.*", "pytest-asyncio", "pytest-mock"],
                "datalib": DATA_LIBRARIES,
                "wandb": [
                    "wandb",
                    *DATA_LIBRARIES,
                ],
                "embeddings": [
                    "scikit-learn>=1.0.2",  # Needed for embedding utils, versions >= 1.1 require python 3.8
                    "tenacity>=8.0.1",
                    "matplotlib",
                    "sklearn",
                    "plotly",
                    *DATA_LIBRARIES,
                ],
            },
            python_requires=">=3.7.1",
            entry_points={
                "console_scripts": [
                    "openai=openai._openai_scripts:main",
                ],
            },
            packages=find_packages(exclude=["tests", "tests.*"]),
            package_data={
                "openai": [
                    "py.typed",
                ]
            },
            author="OpenAI",
            author_email="support@openai.com",
            url="https://github.com/openai/openai-python",
        )

        ######################################################################################
        import openai
        from openai import cli
        openai.api_version = '2022-12-01'
        openai.api_base = '' # Please add your endpoint here
        openai.api_type = 'azure'
        openai.api_key = ''  # Please add your api key here
        # from azure.identity import DefaultAzureCredential

        # default_credential = DefaultAzureCredential()
        # token = default_credential.get_token("https://cognitiveservices.azure.com/.default")

        # openai.api_type = 'azure_ad'
        # openai.api_key = token.token

        import shutil
        import json

        training_file_name = 'training.jsonl'
        validation_file_name = 'validation.jsonl'

        sample_data = [{"prompt": "When I go to the store, I want an", "completion": "apple."},
            {"prompt": "When I go to work, I want a", "completion": "coffee."},
            {"prompt": "When I go home, I want a", "completion": "soda."}]

        print(f'Generating the training file: {training_file_name}')
        with open(training_file_name, 'w') as training_file:
            for entry in sample_data:
                json.dump(entry, training_file)
                training_file.write('\n')

        print(f'Copying the training file to the validation file')
        shutil.copy(training_file_name, validation_file_name)

        print('Checking for existing uploaded files.')
        results = []
        files = openai.File.list().data
        print(f'Found {len(files)} total uploaded files in the subscription.')
        for item in files:
            if item["filename"] in [training_file_name, validation_file_name]:
                results.append(item["id"])
        print(f'Found {len(results)} already uploaded files that match our names.')

        print(f'Deleting already uploaded files...')
        for id in results:
            openai.File.delete(sid = id)

        import time

        def check_status(training_id, validation_id):
            train_status = openai.File.retrieve(training_id)["status"]
            valid_status = openai.File.retrieve(validation_id)["status"]
            print(f'Status (training_file | validation_file): {train_status} | {valid_status}')
            return (train_status, valid_status)

        #importing our two files
        training_id = cli.FineTune._get_or_upload(training_file_name, True)
        validation_id = cli.FineTune._get_or_upload(validation_file_name, True)

        #checking the status of the imports
        (train_status, valid_status) = check_status(training_id, validation_id)

        while train_status not in ["succeeded", "failed"] or valid_status not in ["succeeded", "failed"]:
            time.sleep(1)
            (train_status, valid_status) = check_status(training_id, validation_id)

        print(f'Downloading training file: {training_id}')
        result = openai.File.download(training_id)
        print(result.decode('utf-8'))

        create_args = {
            "training_file": training_id,
            "validation_file": validation_id,
            "model": "babbage",
            "compute_classification_metrics": True,
            "classification_n_classes": 3,
            "n_epochs": 20,
            "batch_size": 3,
            "learning_rate_multiplier": 0.3
        }
        resp = openai.FineTune.create(**create_args)
        job_id = resp["id"]
        status = resp["status"]

        print(f'Fine-tunning model with jobID: {job_id}.')

        import signal
        import datetime

        def signal_handler(sig, frame):
            status = openai.FineTune.retrieve(job_id).status
            print(f"Stream interrupted. Job is still {status}.")
            return

        print(f'Streaming events for the fine-tuning job: {job_id}')
        signal.signal(signal.SIGINT, signal_handler)

        events = openai.FineTune.stream_events(job_id)
        try:
            for event in events:
                print(f'{datetime.datetime.fromtimestamp(event["created_at"])} {event["message"]}')

        except Exception:
            print("Stream interrupted (client disconnected).")

        status = openai.FineTune.retrieve(id=job_id)["status"]
        if status not in ["succeeded", "failed"]:
            print(f'Job not in terminal status: {status}. Waiting.')
            while status not in ["succeeded", "failed"]:
                time.sleep(2)
                status = openai.FineTune.retrieve(id=job_id)["status"]
                print(f'Status: {status}')
        else:
            print(f'Finetune job {job_id} finished with status: {status}')

        print('Checking other finetune jobs in the subscription.')
        result = openai.FineTune.list()
        print(f'Found {len(result.data)} finetune jobs.')

        #Fist let's get the model of the previous job:
        result = openai.FineTune.retrieve(id=job_id)
        if result["status"] == 'succeeded':
            model = result["fine_tuned_model"]

        # Now let's create the deployment
        print(f'Creating a new deployment with model: {model}')
        result = openai.Deployment.create(model=model, scale_settings={"scale_type":"standard"})
        deployment_id = result["id"]

        print(f'Checking for deployment status.')
        resp = openai.Deployment.retrieve(id=deployment_id)
        status = resp["status"]
        print(f'Deployment {deployment_id} is with status: {status}')

        print('While deployment running, selecting a completed one.')
        deployment_id = None
        result = openai.Deployment.list()
        for deployment in result.data:
            if deployment["status"] == "succeeded":
                deployment_id = deployment["id"]
                break

        if not deployment_id:
            print('No deployment with status: succeeded found.')
        else:
            print(f'Found a successful deployment with id: {deployment_id}.')


        print('Sending a test completion job')
        start_phrase = 'When I go home, I want a'
        response = openai.Completion.create(deployment_id=deployment_id, prompt=start_phrase, temperature=0, stop=".")
        text = response['choices'][0]['text'].replace('\n', '').replace(' .', '.').strip()
        print(f'"{start_phrase} {text}."')

        print(f'Deleting deployment: {deployment_id}')
        openai.Deployment.delete(sid=deployment_id)


#Parler losqu'il ne comprend pas        
    else:
        parler('je ne comprends pas')
while True:
    lancer_assistant()
