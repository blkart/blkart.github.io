---
layout: post
title: Glance组件之深入探索『3』——源码框架分析
date: 2015-03-28
categories: blog
tags: [openstack,glance]
description: Glance组件之深入探索

---

PS:分析中如有错误，请各位看官指正。

* * *

两篇的介绍后，这篇来深入到代码里看看。

Glance组件的代码托管在GitHub上，包括两个项目：[glance](https://github.com/openstack/glance)、[glance_store](https://github.com/openstack/glance_store)(store adapter)，有兴趣可以clone下来深入研究。

* * *

前面提到一个服务glance-api，下面就以它为例，先来看下服务的启动流程：

    # filename: glance/cmd/api.py
    def main():
        try:
            # config.parse_args()中会确认要加载的配置文件的绝对路径
            # 并且将配置文件中的配置存放到config.CONF中供后续调用
            config.parse_args()    
            # 设置eventlet的hub，配置文件中没有该配置选项，代码中默认值为poll
            wsgi.set_eventlet_hub()
            log.setup('glance')

            # 初始化后端存储，注册与后端存储（filesystem、eqlx、ceph等）相关的控制模块，
            # 并校验默认存储方式。
            glance_store.register_opts(config.CONF)
            glance_store.create_stores(config.CONF)    
            glance_store.verify_default_store()

            # 确认是否启用OpenStack分析功能
            if cfg.CONF.profiler.enabled:
                _notifier = osprofiler.notifier.create("Messaging",
                                                       notifier.messaging, {},
                                                       notifier.get_transport(),
                                                       "glance", "api",
                                                       cfg.CONF.bind_host)
                osprofiler.notifier.set(_notifier)
            else:
                osprofiler.web.disable()

            # 通过wsgi.Server()创建一个http服务器（使用了eventlet绿色线程(协程)），
            # 调用start()方法启动服务器。
            # server.wait()让server等待接收http rest查询。
            server = wsgi.Server()
            server.start(config.load_paste_app('glance-api'), default_port=9292)
            server.wait()

具体看下server这一部分。 首先通过wsgi.Server实例化了一个server，然后通过server.start()启动server。其中通过config.load_paste_app('glance-api')获得一个要加载的app。

先看下config.load_paste_app()函数相关代码，该函数最终返回一个app作为server.start()函数的application参数：

    # filename glance/common/config.py 
    def load_paste_app(app_name, flavor=None, conf_file=None): 
        # 根据配置文件(glance-api.conf)的配置，确定最终的app_name，
        # 例如配置文件中flavor=keystone，则app_name=glance-api-keystone
        app_name += _get_deployment_flavor(flavor)    

        if not conf_file:
            # 获取paste的配置文件的绝对路径
            conf_file = _get_deployment_config_file()    

        try:
            logger = logging.getLogger(__name__)
            logger.debug("Loading %(app_name)s from %(conf_file)s",
                         {'conf_file': conf_file, 'app_name': app_name})

            # 调用paste.deploy中的loadapp()函数加载paste配置文件中的app_name对应的app
            # (例如glance-api-keystone)
            app = deploy.loadapp("config:%s" % conf_file, name=app_name)    

            return app

    def _get_deployment_flavor(flavor=None):
        if not flavor:
            # 读取配置文件(glance-api.conf)中[paste_deploy]配置块中flavor参数的配置。
            # from oslo.config import cfg  ==&gt;  CONF = cfg.CONF
            # 这就属于oslo.config项目的内容了，有时间单开一篇介绍细节部分
            flavor = CONF.paste_deploy.flavor    
        # 如果配置文件中配置了flavor，则返回'-参数内容'，
        # 例如配置文件中flavor=keystone，则返回'-keystone'
        return '' if not flavor else ('-' + flavor)   

    def _get_deployment_config_file():
        # 读取配置文件(glance-api.conf)中[paste_deploy]配置块中config_file参数的配置。
        path = CONF.paste_deploy.config_file    
        if not path:
            # 如果配置文件中未配置config_file参数，则根据program名字生成path
            path = _get_paste_config_path()    
        if not path:
            msg = _("Unable to locate paste config file for %s.") % CONF.prog
            raise RuntimeError(msg)
        return os.path.abspath(path)

这部分涉及到paste.deploy的相关内容，可以参考以下内容：

* [python paste.deploy 探索](http://mathslinux.org/?p=596)

向外继续来看下server.start(self, application, default_port)

    # filename: glance/common/wsgi.py
    def start(self, application, default_port):
        """
        Run a WSGI server with the given application.

        :param application: The application to be run in the WSGI server
        :param default_port: Port to bind to if none is specified in conf
        """
        self.application = application
        self.sock = get_socket(default_port)
        if CONF.workers == 0:
            # Useful for profiling, test, debug etc.
            self.pool = self.create_pool()
            self.pool.spawn_n(self._single_run, self.application, self.sock)
            return
        else:
            self.logger.info(_("Starting %d workers") % CONF.workers)
            signal.signal(signal.SIGTERM, kill_children)
            signal.signal(signal.SIGINT, kill_children)
            signal.signal(signal.SIGHUP, hup)
            while len(self.children) &lt; CONF.workers:
                self.run_child()

    def run_child(self):
        pid = os.fork()
        if pid == 0:
            signal.signal(signal.SIGHUP, signal.SIG_DFL)
            signal.signal(signal.SIGTERM, signal.SIG_DFL)
            # ignore the interrupt signal to avoid a race whereby
            # a child worker receives the signal before the parent
            # and is respawned unnecessarily as a result
            signal.signal(signal.SIGINT, signal.SIG_IGN)
            self.run_server()
            self.logger.info(_('Child %d exiting normally') % os.getpid())
            # self.pool.waitall() has been called by run_server, so
            # its safe to exit here
            sys.exit(0)
        else:
            self.logger.info(_('Started child %s') % pid)
            self.children.append(pid)

    def run_server(self):
        """Run a WSGI server."""
        if cfg.CONF.pydev_worker_debug_host:
            utils.setup_remote_pydev_debug(cfg.CONF.pydev_worker_debug_host,
                                           cfg.CONF.pydev_worker_debug_port)

        eventlet.wsgi.HttpProtocol.default_request_version = "HTTP/1.0"
        #创建一个eventlet协程池
        self.pool = self.create_pool()
        try:
            #将application放到eventlet协程池里运行，start动作完成。
            eventlet.wsgi.server(self.sock,
                                 self.application,
                                 log=logging.WritableLogger(self.logger),
                                 custom_pool=self.pool,
                                 debug=False)
        except socket.error as err:
            if err[0] != errno.EINVAL:
                raise
        self.pool.waitall()

    def create_pool(self):
        return eventlet.GreenPool(size=self.threads)

将application放到协程池里运行。这部分里面涉及到了eventlet里面的协程（GreenThread绿色线程），可以参考以下内容：

* [Coroutine(协程) 介绍](http://mathslinux.org/?p=234)
* [eventlet官网文档](http://eventlet.net/)。

* * *

前面这部分涉及到了两种配置文件的读取：

* glance-api.conf：用户配置文件
* glance-api-paste.ini：WSGI 配置文件，

用户配置文件(glance-api.conf)的解析是通过glance.common.config.parse_args()实现的，具体来看下

    # filename: glance/common/config.py
    # 从oslo.config中导入cfg
    from oslo.config import cfg

    # 定义名为“paste_deploy”的参数组，其中包含两个选项：flavor、config_file
    # 这两个参数对应了glance-api.conf配置文件里的以下部分的配置：
    # [paste_deploy]
    # flavor=
    # config_file =
    paste_deploy_opts = [
        cfg.StrOpt('flavor',
                   help=_('Partial name of a pipeline in your paste configuration '
                          'file with the service name removed. For example, if '
                          'your paste section name is '
                          '[pipeline:glance-api-keystone] use the value '
                          '"keystone"')),
        cfg.StrOpt('config_file',
                   help=_('Name of the paste configuration file.')),
    ]
    CONF = cfg.CONF
    # 注册多个选项
    CONF.register_opts(paste_deploy_opts, group='paste_deploy')

    # glance/cmd/api.py 中的config.parse_args() 部分调用该方法找到配置文件，
    # 并将配置文件的内容加载到CONF中，需要获取某个配置项时，
    # 通过CONF.groupname.option调用即可，
    # 可以参考_get_deployment_flavor()中的例子
    def parse_args(args=None, usage=None, default_config_files=None):
        CONF(args=args,
             project='glance',
             version=version.cached_version_string(),
             usage=usage,
             default_config_files=default_config_files)

    def _get_deployment_flavor(flavor=None):
        if not flavor:
            # 当需要获取配置文件中的flavor配置项时,
            # 调用CONF.paste_deploy.flavor即可
            flavor = CONF.paste_deploy.flavor
        return '' if not flavor else ('-' + flavor)

以上这段代码截选自glance/common/config.py，对于组件配置文件(XXX.conf)的解析，glance并没有自己实现配置文件解析功能，而是引入了oslo.conf模块，该模块被所有OpenStack组件引用用于实现对配置文件的解析。

想要自己尝试写个例子，可以研究下[这篇blog](http://mathslinux.org/?p=620)

WSGI 配置文件(glance-api-paste.ini)的解析是通过paste模块的deploy.loadapp()方法实现的，关于paste的deploy.loadapp()具体如何实现文件的解析不在这里展开，如果不熟悉paste模块可以看下这篇blog：

* [python paste.deploy 探索](http://mathslinux.org/?p=596)

下面来分析下glance-api-paste.ini文件的部分内容

    # Use this pipeline for keystone auth
    [pipeline:glance-api-keystone]
    pipeline = versionnegotiation osprofiler authtoken context  rootapp

    [composite:rootapp]
    paste.composite_factory = glance.api:root_app_factory
    /: apiversions
    /v1: apiv1app
    /v2: apiv2app

    [app:apiversions]
    paste.app_factory = glance.api.versions:create_resource

    [app:apiv1app]
    paste.app_factory = glance.api.v1.router:API.factory

    [app:apiv2app]
    paste.app_factory = glance.api.v2.router:API.factory

    # 检查请求中的版本信息，如果请求的不是当前所支持的版本，则返回当前支持的版本信息
    [filter:versionnegotiation]
    paste.filter_factory = glance.api.middleware.version_negotiation:VersionNegotiationFilter.factory

    # 用来enables tracing for an application。似乎是在 enabled = yes 的情况下，记录 request 的一些信息。具体功能需要后续研究。
    [filter:osprofiler]
    paste.filter_factory = osprofiler.web:WsgiMiddleware.factory
    hmac_keys = SECRET_KEY
    enabled = yes

    # 该 filter 会通过校验request所带的 token 来校验用户身份。当request来的时候，它根据 token 来判断这个请求是否合法。校验失败则直接返回 401 错误，校验通过则提取 token 中的信息，放入到env中，供后续的其它app使用。
    [filter:authtoken]
    paste.filter_factory = keystonemiddleware.auth_token:filter_factory
    delay_auth_decision = true

    # 将认证信息保存到context中，供后续app调用。
    [filter:context]
    paste.filter_factory = glance.api.middleware.context:ContextMiddleware.factory

简单解释下上面这个文件中的几个术语：

* app：实际处理REST API请求的python类。

* filter：过滤器，为app提供一层封装，在app处理请求之前会先调用filter的对象。

* pipeline：所对应的对象是对filter和app的的封装，它将多个filter和某个app绑在一起，在app处理请求之前要先通过pipline指定的在app之前的filter的处理。

* composite：用来完成将将一个请求调度定向(dispatched)到多个(多种)应用上. 比如应用有 v1, v2 的版本, 就可以用 composite 来做调度. composite 其实像是 app, 但是实际上是由 多个应用组成.

在前面对glance/common/config.py的分析中已经介绍过load_paste_app(app_name, flavor=None, conf_file=None)函数，该函数会调用deploy.loadapp("config:%s" % conf_file, name=app_name)解析conf_file文件，加载app_name对应的pipeline。

举个例子来说，如果我们在配置文件glance-api.conf文件中配置了flavor=keystone，此时app_name=glance-api-keystone，那么deploy.loadapp()会到glance-api-paste.ini文件中查找值为glance-api-keystone的pipeline或app，从上面的配置文件中，我们可以找到一个名为glance-api-keystone的pipeline，此时deploy.loadapp()会依次加载versionnegotiation、osprofiler、authtoken、context过滤器及最后的rootapp。

glance-api服务加载完filter和app后，即可响应REST API请求。

* * *

OK，下面了解下glance-api是如何处理REST API请求的。 前面我们看到glance-api服务加载了versionnegotiation、osprofiler、authtoken、context过滤器及一个rootapp，这里有个请求处理顺序的问题，当server接收到一个请求时，rootapp会依次调用前面的filter(过滤器)对请求进行处理，在处理过程中，如果某个filter的判断条件不满足，直接将filter反馈信息响应给客户端，不再将请求交给下个模块进行处理。具体的流程如下图所示：

![选区_095](/img/blog_img/095.png)

当请求经过前面各个filter的处理后，最终来到rootapp，rootapp会根据请求信息将请求调度到对应的app，比如请求的是v1版本，rootapp会调度apiv1app这个app来处理请求。

apiv1app和apiv2app两个app的工厂方法的 API 类都继承自 wsgi.Router, Router 类利用了 python-routes 模块做请求信息URL的选择处理, 以v1版本为例，看下具体流程图：

![选区_098](/img/blog_img/098.png)

* * *

OK，源码框架大致是这样的。具体一些的功能的实现源码分析后续再写。
