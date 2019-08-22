# UED开发编码规范

本章节是对UED规范的编码规范补充，请结合`PC-UEDGUIDE.pdf`规范查看。

## 表格查询页面

大多数表格查询页面由上方的查询表单及下方的表格组成，对于常见的超过3个查询条件的表单编码示例如下：

注意：代码中引用的常量均来自`utils/constants`文件中定义的各类布局常用值，可在本文档[页面布局常量](../index_constants)中查看。
```
       ...
       <Form layout="inline" className="more-fields-search-form">
        <Row {...SEARCH_FORM_ROW_LAYOUT}>
          <Col {...FORM_COL_4_LAYOUT}>
            <Form.Item {...SEARCH_FORM_ITEM_LAYOUT} label={intl.get(`${prefix}.code`).d('8D编号')}>
              {getFieldDecorator('problemNum', {
                rules: [
                  {
                    max: 20,
                    message: intl.get('hzero.common.validation.max', {
                      max: 20,
                    }),
                  },
                ],
              })(<Input trim typeCase="upper" inputChinese={false} />)}
            </Form.Item>
          </Col>
          <Col {...FORM_COL_4_LAYOUT}>
            <Form.Item {...SEARCH_FORM_ITEM_LAYOUT} label={intl.get(`${prefix}.title`).d('8D标题')}>
              {getFieldDecorator('problemTitle', {
                rules: [
                  {
                    max: 80,
                    message: intl.get('hzero.common.validation.max', {
                      max: 80,
                    }),
                  },
                ],
              })(<Input trim />)}
            </Form.Item>
          </Col>
          <Col {...FORM_COL_4_LAYOUT}>
            <Form.Item
              {...SEARCH_FORM_ITEM_LAYOUT}
              label={intl.get(`${prefix}.issue`).d('问题类型')}
            >
              {getFieldDecorator('problemTypeCode', {})(
                <Select allowClear>
                  {issueType.map(item => (
                    <Select.Option key={item.value} value={item.value}>
                      {item.meaning}
                    </Select.Option>
                  ))}
                </Select>
              )}
            </Form.Item>
          </Col>
          <Col {...FORM_COL_4_LAYOUT} className="search-btn-more">
            <Form.Item>
              <Button onClick={this.toggleForm}>
                {expandForm
                  ? intl.get('hzero.common.button.collected').d('收起查询')
                  : intl.get('hzero.common.button.viewMore').d('更多查询')}
              </Button>
              <Button data-code="reset" onClick={this.handleFormReset}>
                {intl.get('hzero.common.button.reset').d('重置')}
              </Button>
              <Button
                data-code="search"
                type="primary"
                htmlType="submit"
                onClick={this.handleSearch}
              >
                {intl.get('hzero.common.button.search').d('查询')}
              </Button>
            </Form.Item>
          </Col>
        </Row>
        ...
```

## 详情表单页面
详情或编辑页面的表单多由`Collapse`组件与`Form`组件相结合。

`Collapse`组件编码示例如下：

```
        ...
        /**
        * onCollapseChange - 折叠面板onChange
        * @param {Array<string>} collapseKeys - Panels key
        */
        @Bind()
        onCollapseChange(collapseKeys) {
            this.setState({
            collapseKeys,
            });
        }
        ...
        ...
        <Content className={classNames(styles['page-content'])}>
          <Spin
            spinning={newFlag ? false : loading.fetch}
            wrapperClassName={classNames(DETAIL_DEFAULT_CLASSNAME)}
          >
            <Collapse
              forceRender
              className="form-collapse"
              defaultActiveKey={collapseKeys}
              onChange={this.onCollapseChange}
            >
              <Collapse.Panel
                showArrow={false}
                header={
                  <Fragment>
                    <h3>{intl.get(`${prefix}.panel.B`).d('基本信息')}</h3>
                    <a>
                      {collapseKeys.includes('B')
                        ? intl.get(`hzero.common.button.up`).d('收起')
                        : intl.get(`hzero.common.button.expand`).d('展开')}
                    </a>
                    <Icon type={collapseKeys.includes('B') ? 'up' : 'down'} />
                  </Fragment>
                }
                key="B"
              >
                <BasicInfoForm {...basicInfoProps} tenantId={this.props.tenantId} />
              </Collapse.Panel>
              ...
```

`Form`组件编码示例如下, `Row`的类名根据该行是编辑表单/展示表单来决定，可在`utils/constants`文件中查看所有常用类名。
```
      ...
      <Form>
        <Row {...EDIT_FORM_ROW_LAYOUT} className="writable-row">
          <Col {...FORM_COL_3_LAYOUT}>
            <Form.Item label={intl.get(`${prefix}.code`).d('8D编号')} {...EDIT_FORM_ITEM_LAYOUT}>
              {getFieldDecorator('problemNum', {
                initialValue: dataSource.problemNum,
              })(<Input disabled />)}
            </Form.Item>
          </Col>
          <Col {...FORM_COL_3_LAYOUT}>
            <Form.Item label={intl.get(`${prefix}.status`).d('8D状态')} {...EDIT_FORM_ITEM_LAYOUT}>
              {getFieldDecorator('problemStatusMeaning', {
                initialValue: newFlag ? statusMeaning : dataSource.problemStatusMeaning,
              })(<Input disabled />)}
            </Form.Item>
          </Col>
          <Col {...FORM_COL_3_LAYOUT}>
            <Form.Item
              label={intl.get(`entity.roles.creator`).d('创建人')}
              {...EDIT_FORM_ITEM_LAYOUT}
            >
              {getFieldDecorator('createdName', {
                initialValue: newFlag ? user : dataSource.createdName,
              })(<Input disabled />)}
            </Form.Item>
          </Col>
        </Row>
        ...
```



