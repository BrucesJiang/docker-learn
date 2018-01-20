FROM python:3.4

# 添加用户组以及用户
RUN groupadd -r uwsgi && useradd -r -g uwsgi uwsgi
RUN pip install Flask==0.10.1 uWSGI==2.0.8 requests==2.5.1 redis==2.10.3
# 创建工作目录
WORKDIR /app
# 将项目放进容器
COPY app /app
# 将脚本放进容器
COPY cmd.sh /

EXPOSE 9090 9191
USER uwsgi

#CMD ["uwsgi", "--http", "0.0.0.0:9090", "--wsgi-file", "/app/identidock.py", \
#"--callable", "app", "--stats", "0.0.0.0:9191"]
# 从CMD命令调用
CMD ["/cmd.sh"]
